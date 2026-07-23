# Txxxx — <Attack name>

> Copy this file for each new detection writeup.

**Technique:** [Txxxx — Name](https://attack.mitre.org/techniques/Txxxx/)
**Tactic:** <e.g. Credential Access>
**Source:** Kali Linux · **Target:** `<host>` (`<ip>`) · **Ingestion path:** <agent 1514 | syslog 514>
**Date run:** <YYYY-MM-DD>

## Objective

What real adversary behavior this simulates, and why an environment would want to catch it.

## Execution

```bash
<exact command run>
```

<What the attack did from the attacker's side — succeeded, failed, credentials found, etc.>

## Detection

| Rule ID | Level | Description | Count |
|---|---|---|---|
| | | | |

<!-- TODO: add screenshot, then uncomment:
![alert screenshot](../assets/screenshots/<file>.png)
-->
<What the alert says, and what an analyst would notice first.>

## ATT&CK mapping

**Txxxx — Name.** <One or two sentences on what the technique represents and where it sits in an
attack chain.>

## Analyst response (NIST SP 800-61)

- **Detection & analysis:** <what confirms this is real vs. noise>
- **Containment:** <immediate action>
- **Eradication:** <removing the cause>
- **Recovery:** <returning to known good>
- **Post-incident:** <what changes so it's caught earlier or prevented>

## Gaps and notes

<What this detection would miss — slower attacks, different protocols, encrypted channels — and what
tuning or additional telemetry would close the gap.>
