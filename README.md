# Cowrie Honeypot Lab

## Environment
- Honeypot: Kali Linux VM — Cowrie SSH Honeypot (port 2222)
- Attacker 1: Windows 11 Second Laptop (192.168.1.83) — Manual
- Attacker 2: Kali Linux (192.168.1.72) — Hydra automated
- SIEM: Windows 11 Main Laptop — Splunk Enterprise 10.4.0

## Status
- [x] Phase 1: Cowrie installed and configured
- [x] Phase 2: Attack simulation — 3 sessions, 11 commands
- [x] Phase 3: Log analysis and IOC extraction
- [ ] Phase 4: Splunk ingestion of Cowrie JSON logs
- [ ] Phase 5: Full documentation

## Investigations
| Date | Session | Severity | Report |
|------|---------|----------|--------|
| 2026-07-02 | 3 attack sessions captured | High | [View](investigations/2026-07-02-cowrie-attack-sessions.md) |

## Key Findings
- 2 unique attacker IPs identified
- 3 credentials captured (root:admin x2, root:password x1)
- 2 HASSH fingerprints — human vs automated tool distinguished
- 10 commands mapped to MITRE ATT&CK framework
- Persistence attempt via cron job captured
