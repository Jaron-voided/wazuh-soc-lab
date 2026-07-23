# Findings

Security findings from the lab, written the way they would be handed to a customer rather than as
personal notes: what was found, how it was confirmed, what the risk is, and what to do about it —
with the practical option listed alongside the correct one.

Writing them this way is deliberate practice. Identifying a weakness is the easy half; communicating
it so a non-specialist can act on it is the half that determines whether anything changes.

| ID | Finding | Severity | Asset | Status |
|---|---|---|---|---|
| [FIND-001](FIND-001-deprecated-ssh-host-key-algorithms.md) | Deprecated SSH host key algorithms | Medium | Legacy Linux host | Open — accepted (lab) |

## Format

Summary → discovery and evidence → technical detail → risk → recommendations → disposition.
Recommendations are ordered: fix the root cause first, mitigations second, and detection
compensations where the root cause can't be fixed.
