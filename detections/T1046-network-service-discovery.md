# T1046 — Network Service Discovery

**Technique:** [T1046 — Network Service Discovery](https://attack.mitre.org/techniques/T1046/)
**Tactic:** Discovery
**Source:** Kali Linux · **Targets:** lab victim addresses (specific hosts, never a range)
**Date run:** <!-- TODO: fill in -->

## Objective

Enumerating listening services and their versions — what an attacker does immediately after gaining
network access, to decide what to attack next. Also the phase that produced
[FIND-001](../findings/FIND-001-deprecated-ssh-host-key-algorithms.md).

## Execution

```bash
# TODO: paste the exact commands run, e.g.
nmap -sV <target-ip>
nmap -sV --script ssh2-enum-algos <target-ip>
```

<!-- TODO: summarize what was found — open ports, service versions, anything notably outdated -->

## Detection

Scanning is a genuinely hard detection problem for a host-based SIEM: a port scan that only touches
listening ports may generate no host-level log entries at all. Expect this one to demonstrate a
**coverage gap** as much as a detection.

| Rule ID | Level | Description | Count |
|---|---|---|---|
| <!-- TODO --> | | | |

<!-- TODO: add screenshot, then uncomment:
![alerts](../assets/screenshots/t1046-alerts.png)
-->
<!-- TODO: add screenshot, or document that no alerts fired -->

<!-- TODO: honest record of what alerted and what didn't. "No alert" is a legitimate and useful
result here — document it rather than hunting for something to show. -->

## ATT&CK mapping

**T1046 — Network Service Discovery.** Identifying services running on remote hosts to find
exploitable targets. Sits under Discovery, typically after initial access and before exploitation.
Adversaries and legitimate administrators do this identically, which is exactly why it's hard to
alert on without context.

## Analyst response (NIST SP 800-61)

- **Detection & analysis:** Determine whether the scan source is authorized. Context decides this,
  not the traffic — a vulnerability scanner and an attacker generate similar packets. Scope which
  hosts were touched and what was exposed.
- **Containment:** If unauthorized, block the source and check whether scanning was followed by
  exploitation attempts against a service the scan identified.
- **Eradication / Recovery:** Nothing to remove from a scan itself; the response is reducing what it
  found — close unnecessary services, patch what the version output exposed.
- **Post-incident:** Reduce attack surface, segment management interfaces, and add network-level
  detection where host logs are blind.

## Gaps and notes

- Host-based logging is largely blind to reconnaissance that doesn't complete connections. Closing
  this properly needs network-layer visibility — an IDS or flow data — rather than SIEM rule tuning.
- Documenting a *miss* is the point of this writeup. Knowing where a detection stack is blind is
  worth more than a clean screenshot.
