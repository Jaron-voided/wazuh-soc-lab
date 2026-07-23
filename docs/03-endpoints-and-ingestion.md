# Phase 3 — Hardening, Enrollment, and Ingestion

A SIEM with nothing reporting into it is a dashboard. This phase hardens the deployment and gets
every endpoint shipping telemetry — which turned out to mean two different ingestion paths, because
one host is too old to support the modern one.

## 3.1 Rotate the default credentials

The stack ships with a publicly documented default password. Rotating it comes before enrollment,
not after — anything enrolled against a known-credential SIEM has to be re-trusted later. Maps to
**CIS Control 5.2**.

Order matters:

1. Log out of the dashboard and close the tab (stale session cookies cause confusing failures).
2. Bring the stack down — **without** the volume flag, which would destroy the volumes, certificates,
   and indices along with it.
3. Generate a bcrypt hash with the indexer image's own hashing tool. The hash is exactly 60
   characters and its alphabet includes `.` and `/`, so a trailing period is part of the hash, not
   punctuation ([I19](issues-log.md#i19--password-hash-appeared-truncated)).
4. Place the hash in the internal users file and remove **every** plaintext occurrence from the
   compose file. Verify with a grep that returns nothing — "I think I got them all" is not
   verification.
5. Bring the stack up, wait for the indexer to go healthy, then reapply the security configuration
   from *inside* the indexer container, where the directory layout is flattened relative to what the
   documentation implies ([I20](issues-log.md#i20--security-tooling-reports-config-directory-not-found)).

Verify with the credential-**prompt** form of `curl` — never by passing the password as an argument,
where it lands in shell history and in the process table visible to other users on the box:

```bash
curl -sk -u admin https://localhost:9200/_cluster/health?pretty   # prompts; new password succeeds
```

Confirm the old password now fails, and confirm dashboard login in a fresh private window so a
cached session can't produce a false pass.

## 3.2 Modern endpoints — agent enrollment

Windows 11 and the Ubuntu SIEM host both run the Wazuh agent, reporting over TCP 1514 on an
encrypted, authenticated channel. The Windows agent is deployed from the dashboard's enrollment
wizard and registered against the manager's address; the Ubuntu host enrolls itself so the SIEM
monitors its own authentication logs.

Self-monitoring the SIEM host produces extra telemetry noise, since it also observes the container
runtime — expected, and worth knowing before it looks like a problem.

If an agent reports Disconnected, test network reachability to the manager port before touching the
agent configuration. Confirm both show Active, then snapshot each endpoint in its clean enrolled
state — that snapshot is the baseline every attack simulation rolls back to.

## 3.3 The legacy host — why it gets syslog instead

Metasploitable 2 is Ubuntu 8.04. Its OpenSSL predates TLS 1.2 and its glibc is far older than any
current build target, so the agent can neither download (the TLS handshake fails against the package
host) nor install (the binary won't run). This isn't a configuration problem to solve — it's a
capability boundary.

The options were: leave the host unmonitored, or accept a lesser ingestion path. Losing visibility
on the most vulnerable machine in the lab is the worse outcome, so the host forwards its syslog to
the manager over UDP 514 instead.

Manager side, this means declaring a syslog listener restricted to the lab subnet
([`configs/wazuh-manager-syslog.xml`](../configs/wazuh-manager-syslog.xml)) and publishing the port
on the manager container. Host side, it's a single forwarding rule in the legacy syslog
configuration ([`configs/metasploitable-syslog.conf`](../configs/metasploitable-syslog.conf)) and a
service restart.

Two traps here, both worth generalizing:

- **The config edit may not reach the running manager.** The live configuration lives in a
  persistent volume seeded only at first container creation, so editing the host-side template and
  cycling the stack silently does nothing. Verify inside the container and confirm the listener
  appears in the manager log ([I23](issues-log.md#i23--manager-configuration-edits-not-taking-effect)).
- **In-container socket tools report no listener even when one exists** — a false negative from
  stripped tooling ([I26](issues-log.md#i26--in-container-socket-tools-show-no-listener)).

### Proving delivery

A test message sent with `logger` never appeared in the dashboard. That looks like failure, and
isn't: the message matches no decoder, and with full logging disabled it's never indexed. The
dashboard can answer "did this alert" — it cannot answer "did this arrive."

So the question got asked at the layer that can answer it, with a packet capture on the host
interface:

```bash
sudo tcpdump -i <lab-interface> -n udp port 514 -c 5
# datagrams from the legacy host to the manager confirm delivery on the wire
```

([I25](issues-log.md#i25--syslog-test-not-visible-in-discover)) Real attack traffic — failed SSH
authentication, for instance — matches rules and alerts normally.

## 3.4 Accessing the legacy host, and the finding that came out of it

Modern SSH clients refuse to connect to this host outright: it offers only SHA-1-based host key
algorithms that current OpenSSH removed from its defaults. Connecting requires re-enabling those
algorithms **per connection**, scoped to that host — never globally, which would quietly weaken
every SSH connection made from that client
([`configs/ssh-legacy-host.md`](../configs/ssh-legacy-host.md)).

That refusal is a security control doing its job, and it's also evidence. It's written up as
[FIND-001](../findings/FIND-001-deprecated-ssh-host-key-algorithms.md) rather than treated as an
obstacle to work around.

## Result — two ingestion tiers

| Endpoint | Path | Transport | Status |
|---|---|---|---|
| `win11-victim` | Agent | TCP 1514, encrypted and authenticated | Active |
| `ubuntu-siem-host` | Agent (self-monitoring) | TCP 1514, encrypted and authenticated | Active |
| `metasploitable` | Syslog forwarding | UDP 514, cleartext, unauthenticated | Delivery confirmed on the wire |

The legacy host doesn't appear in the agent inventory at all — it has no agent, so it has no agent
health, no file integrity monitoring, and no ability to prove the log source is who it claims to be.
Its traffic is trusted only because it's confined to a lab segment; on a production network the same
path would need segmentation and compensating monitoring around it.

That mix — strong telemetry where the platform allows it, degraded telemetry where it doesn't — is
the realistic state of most environments, and knowing which tier a given alert came from changes how
much weight it carries.

**Next:** [detections](../detections/) — attacks, the alerts they produced, and ATT&CK mappings.
