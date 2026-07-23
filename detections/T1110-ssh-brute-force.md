# T1110 — SSH Brute Force

**Technique:** [T1110 — Brute Force](https://attack.mitre.org/techniques/T1110/)
(sub-technique [T1110.001 — Password Guessing](https://attack.mitre.org/techniques/T1110/001/))
**Tactic:** Credential Access
**Source:** Kali Linux · **Target:** Metasploitable 2 · **Ingestion path:** syslog, UDP 514
**Date run:** <!-- TODO: fill in -->

## Objective

Repeated authentication attempts against an exposed SSH service — one of the most common ways an
external attacker gets an initial foothold, and one of the few attacks that is loud by nature. If a
SIEM can't catch this, it can't catch much.

This target is the legacy host, so it's also a test of whether the **syslog ingestion tier** produces
usable detections, or only proves that packets arrive.

## Execution

```bash
# TODO: paste the exact command run, e.g.
hydra -l <user> -P <wordlist> ssh://<target-ip>
```

<!-- TODO: what happened from the attacker's side — how many attempts, did any credential succeed,
how long did it take -->

## Detection

Authentication failures arrive as syslog events and are evaluated by the SSH rule set. Expect a
volume of individual failure events plus a correlated rule that fires once a threshold is crossed
within a time window — the correlated one is the actual detection; the individual failures are the
evidence.

| Rule ID | Level | Description | Count |
|---|---|---|---|
| <!-- TODO --> | | Attempted login with a non-existent user | |
| <!-- TODO --> | | Multiple authentication failures — possible brute force | |

<!-- Commonly rules 5710 and 5712 in the default SSH ruleset — confirm the IDs and levels actually
observed in the dashboard rather than assuming. -->

<!-- TODO: add screenshot, then uncomment:
![Wazuh alerts for SSH brute force](../assets/screenshots/t1110-alerts.png)
-->
<!-- TODO: add screenshot -->

<!-- TODO: what the alert view shows — source IP, target user(s), time clustering -->

## ATT&CK mapping

**T1110 — Brute Force.** Systematically guessing credentials to gain access to an account. It sits
early in an intrusion, under Credential Access, and typically precedes lateral movement or
persistence. The detection signature is volume and rate: many failures from one source in a short
window, often against multiple usernames.

## Analyst response (NIST SP 800-61)

- **Detection & analysis:** Confirm the source address, the accounts targeted, and — the question
  that determines severity — whether any attempt **succeeded**. A successful authentication
  immediately following a burst of failures escalates this from noise to an active incident.
- **Containment:** Block the source address at the network boundary; disable or force a reset on
  any account that authenticated successfully.
- **Eradication:** Verify no persistence was established — new accounts, modified authorized keys,
  scheduled tasks, or altered service configuration.
- **Recovery:** Restore known-good configuration; rotate credentials for affected accounts.
- **Post-incident:** Rate-limit or key-only SSH authentication, restrict management access by source
  address, and alert on successful authentication that closely follows repeated failures.

## Gaps and notes

- The syslog tier is unauthenticated and cleartext — the log source can't prove it is who it claims
  to be. Detections from this tier are directionally useful but not forensically strong.
- A slow, distributed brute force spread across sources and hours would stay under a
  threshold-and-window rule. Catching that needs behavioral baselining, not counting.
- <!-- TODO: anything the attack did that produced no alert at all -->
