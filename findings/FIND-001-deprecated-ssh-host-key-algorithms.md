# FIND-001 — Deprecated SSH Host Key Algorithms on Legacy Host

| | |
|---|---|
| **Finding ID** | FIND-001 |
| **Severity** | Medium |
| **Status** | Open — accepted risk (lab environment) |
| **Affected asset** | `metasploitable` — legacy Linux host, lab segment |
| **Category** | Weak cryptography / outdated software |
| **Identified** | During endpoint onboarding (Phase 3) |

## Summary

The affected host's SSH service offers only host key algorithms based on SHA-1, which current
OpenSSH releases have removed from their default set. Any client built in the last several years
refuses the connection outright unless the deprecated algorithms are explicitly re-enabled.

The connection failure is what surfaced this. The failure is the control working correctly — but
the condition behind it is a real weakness, so it's recorded as a finding rather than as a
workaround.

## Discovery

Attempting to connect from a current SSH client fails during algorithm negotiation with a
no-matching-host-key error. Confirmed by enumerating the service's supported algorithms directly:

```bash
nmap -sV --script ssh2-enum-algos <target-ip>
```

<!-- TODO: paste the relevant portion of the algorithm output as evidence -->

## Technical detail

SSH host keys authenticate the *server* to the client — they're what protects against
machine-in-the-middle attacks and what makes the client's known-hosts warning meaningful.

SHA-1 has known practical collision attacks, and the SSH ecosystem deprecated SHA-1-based signature
algorithms accordingly. Modern clients therefore refuse them by default. A host offering only these
algorithms cannot present a trustworthy server identity to a current client, which leaves two bad
options: refuse the connection and lose access, or weaken the client to reach the server.

The second option is the real risk. Re-enabling deprecated algorithms **globally** on a client — the
convenient fix, and the one most search results suggest — silently downgrades every SSH connection
that client makes, including to properly configured systems. A one-host problem becomes an
every-connection problem.

## Risk

**Likelihood:** Low in a segmented lab; higher anywhere the host is reachable from untrusted
networks.
**Impact:** Weakened server authentication enables machine-in-the-middle interception of an
administrative session, exposing credentials and command traffic.

The underlying condition is broader than SSH: a host this far out of support receives no security
updates for any component. The SSH weakness is a symptom of an end-of-life system, and the SSH
configuration is not where the actual risk lives.

## Recommendations

**Preferred — remediate the root cause.** Migrate the workload to a supported operating system.
An end-of-life host cannot be patched, so every other control is compensating for a gap that stays
open. Everything below is mitigation, not a fix.

**If the host must remain in service:**

1. Restrict administrative access to a specific management network or jump host, so the weak
   authentication path isn't reachable from general network segments.
2. Segment the host from systems that don't need to reach it, and monitor its traffic accordingly.
3. Where the SSH implementation supports it, enable stronger host key algorithms and disable the
   deprecated ones. On a host of this vintage that may not be possible — which itself supports
   recommendation 1.
4. Scope any client-side compatibility flags to this host alone, per connection or in a host-specific
   client configuration block. **Never apply them globally.**

**Detection compensations, given the host also cannot run a modern monitoring agent:**

5. Alert on successful authentication to this host from any source outside the approved management
   network.
6. Treat its forwarded logs as lower-assurance evidence — the syslog path is unauthenticated, so
   the log source cannot prove its identity. Corroborate anything significant with network-level
   telemetry.

## Lab disposition

Accepted for the lab: the host is intentionally vulnerable, exists to generate attack telemetry, and
is confined to an isolated segment with no external reachability. Client-side compatibility flags
are applied per connection only, never globally — see
[`configs/ssh-legacy-host.md`](../configs/ssh-legacy-host.md).

## References

- [`docs/03-endpoints-and-ingestion.md`](../docs/03-endpoints-and-ingestion.md) — onboarding context
- [Issue I22](../docs/issues-log.md#i22--ssh-refuses-to-connect-to-the-legacy-host) — how it surfaced
- [T1046 — Network Service Discovery](../detections/T1046-network-service-discovery.md) — the
  enumeration that confirmed it
