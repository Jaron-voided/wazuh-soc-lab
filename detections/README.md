# Detections

One writeup per attack, in a consistent shape: what was run, what the SIEM produced, which MITRE
ATT&CK technique it maps to, and how an analyst would respond.

The point isn't that a tool fired an alert. It's the path from adversary action to analyst decision
— including the cases where an attack produced *no* alert, which are worth documenting precisely
because they define the gap.

## Index

| Technique | Attack | Source → Target | Status |
|---|---|---|---|
| [T1110](T1110-ssh-brute-force.md) | SSH brute force (Hydra) | Kali → Metasploitable | Attack run, writeup in progress |
| [T1046](T1046-network-service-discovery.md) | Service and version discovery (Nmap) | Kali → lab targets | Attack run, writeup in progress |
| [T1190](T1190-exploit-public-facing-application.md) | Exploitation of a vulnerable service (Metasploit) | Kali → Metasploitable | Attack run, writeup in progress |
| [T1595.002](T1595-web-application-scanning.md) | Web application scanning (Burp Suite) | Kali → Metasploitable web services | Attack run, writeup in progress |

## Format

Each writeup follows [`TEMPLATE.md`](TEMPLATE.md):

1. **Objective** — what the attack simulates and why it's worth detecting
2. **Execution** — the command run, against which target, at what time
3. **Detection** — the alerts produced, with rule ID, level, and screenshot
4. **ATT&CK mapping** — technique, tactic, and what the technique actually represents
5. **Analyst response** — triage through the NIST SP 800-61 lifecycle
6. **Gaps and notes** — what wasn't caught, and what would improve coverage

## Conventions

- Attacks are directed at specific lab IP addresses only — never a range.
- Each victim is rolled back to its clean enrolled snapshot before an attack, so alerts aren't
  contaminated by prior activity.
- Timestamps in screenshots are lab-local and used only to correlate an attack with its alert.
