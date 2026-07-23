# T1595.002 — Vulnerability Scanning (Web Applications)

**Technique:** [T1595.002 — Active Scanning: Vulnerability Scanning](https://attack.mitre.org/techniques/T1595/002/)
**Tactic:** Reconnaissance
**Source:** Kali Linux (Burp Suite) · **Target:** Metasploitable 2 web services
**Date run:** <!-- TODO: fill in -->

## Objective

Probing a web application for exploitable weaknesses — the reconnaissance that precedes most web
attacks. Web traffic is high-volume and mostly benign, which makes separating a scanner from normal
users the actual challenge.

## Execution

```
# TODO: what was tested and how, e.g. proxied traffic through Burp against <app>,
# ran an active scan against <endpoint>, tested <parameter> for injection
```

<!-- TODO: what the scan surfaced — injection points, outdated components, missing headers -->

## Detection

| Rule ID | Level | Description | Count |
|---|---|---|---|
| <!-- TODO --> | | | |

<!-- TODO: add screenshot, then uncomment:
![alerts](../assets/screenshots/t1595-alerts.png)
-->
<!-- TODO: add screenshot -->

<!-- TODO: whether web server logs were being ingested at all, and what pattern the alerts keyed on
— error rates, suspicious URIs, request volume -->

## ATT&CK mapping

**T1595.002 — Active Scanning: Vulnerability Scanning.** Probing a target for weaknesses that can
be exploited later. It's pre-compromise reconnaissance: nothing has been breached yet, but it is
frequently the earliest visible sign that a target has been selected.

## Analyst response (NIST SP 800-61)

- **Detection & analysis:** Establish whether the source is an authorized scanner. Look at request
  volume, error-response rate, and whether requests follow patterns a human wouldn't produce.
- **Containment:** Rate-limit or block the source if unauthorized; watch for scanning that turns
  into targeted exploitation of a specific finding.
- **Eradication / Recovery:** Nothing to eradicate from reconnaissance — the response is fixing what
  it found before someone acts on it.
- **Post-incident:** Address the weaknesses surfaced, and treat repeat scanning from the same source
  as a targeting signal worth escalating.

## Gaps and notes

- Detection here depends entirely on web server logs being ingested and decoded. Without that, this
  attack is invisible to the SIEM regardless of how noisy it is on the network.
- A low-and-slow scan blended into legitimate traffic would evade volume-based rules — the same
  fundamental limit as [T1110](T1110-ssh-brute-force.md).
