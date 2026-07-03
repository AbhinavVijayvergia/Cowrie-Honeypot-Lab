# Cowrie Honeypot Lab
> A deception-based defense lab deploying Cowrie SSH honeypot to capture, 
> analyze, and detect real attack behavior using Splunk SIEM.

## Lab Architecture
See [architecture/lab-diagram.md](architecture/lab-diagram.md)

**Quick Overview:**
- Honeypot: Kali Linux VM — Cowrie SSH Honeypot (port 2222)
- Attacker: Windows 11 Second Laptop (192.168.1.83) — Manual SSH + Hydra
- SIEM: Windows 11 Main Laptop — Splunk Enterprise 10.4.0

## Tools & Versions
- Cowrie SSH Honeypot — Latest (cloned from GitHub)
- Splunk Enterprise 10.4.0 — Free License
- Hydra v9.6
- Kali Linux 2025.4
- Python 3.13 (Cowrie runtime)

## Repository Structure
    Cowrie-Honeypot-Lab/
    ├── architecture/
    │   └── lab-diagram.md
    ├── setup/
    │   └── 01-cowrie-install.md
    ├── investigations/
    │   └── 2026-07-02-cowrie-attack-sessions.md
    ├── logs/
    │   └── cowrie-session-01.json
    ├── screenshots/
    │   ├── splunk-cowrie-events.png
    │   ├── high-risk-commands.png
    │   ├── credentials-captured.png
    │   ├── hassh-fingerprints.png
    │   ├── cowrie-running-logs.png
    │   └── attacker-inside-honeypot.png
    └── README.md

## Status
- [x] Phase 1: Cowrie installed and configured
- [x] Phase 2: Attack simulation — 3 sessions, 11 commands
- [x] Phase 3: Log analysis and IOC extraction
- [x] Phase 4: Splunk ingestion of Cowrie JSON logs
- [x] Phase 5: Full documentation

## Attack Sessions Captured
| Session ID | Source IP | Type | Credentials | Commands |
|------------|-----------|------|-------------|----------|
| cdb9eb1a776f | 192.168.1.83 | Manual | root:admin | 5 |
| e4338cddb029 | 192.168.1.72 | Hydra | root:password | 0 |
| 2d306b4005f7 | 192.168.1.72 | Hydra | root:password | 0 |
| 93062fb8117a | 192.168.1.83 | Manual | root:admin | 6 |

## IOCs Extracted
| Type | Value |
|------|-------|
| Attacker IP (manual) | 192.168.1.83 |
| Attacker IP (automated) | 192.168.1.72 |
| Credential | root:admin |
| Credential | root:password |
| HASSH (human) | 701158e75b508e76f0410d5d22ef9df0 |
| HASSH (Hydra) | 015322ee8471fa8338c558a918183b11 |
| Malware URL | http://malware.example.com/virus.sh |
| Persistence | */5 * * * * /tmp/backdoor.sh |

## MITRE ATT&CK Coverage
| Tactic | Techniques Observed |
|--------|-------------------|
| Discovery | T1033, T1082, T1016, T1057, T1083, T1087 |
| Execution | T1059 |
| Command & Control | T1105 |
| Persistence | T1053 |

## Detection Rules Built
| Rule | Trigger | Splunk Alert |
|------|---------|--------------|
| High Risk Command Detection | wget, curl, chmod, crontab in honeypot sessions | Active |
| Honeypot Login Detection | Any successful login to honeypot | Active |

## Key Findings
- Cowrie successfully deceived attacker — real shell behavior simulated
- HASSH fingerprinting distinguished human attacker from Hydra tool
- Attacker followed textbook post-exploitation sequence
- Persistence attempt via cron job fully captured
- Malware download URL logged despite failed DNS resolution
- Attacker typo (crontab -1) confirmed human operator

## Investigations
| Date | Description | Severity | Report |
|------|-------------|----------|--------|
| 2026-07-02 | 3 attack sessions, 11 commands, persistence attempt | High | [View](investigations/2026-07-02-cowrie-attack-sessions.md) |

## Screenshots
| Description | File |
|-------------|------|
| Cowrie events by type in Splunk | [View](screenshots/splunk-cowrie-events.png) |
| High risk command detection | [View](screenshots/high-risk-commands.png) |
| Credentials captured in Splunk | [View](screenshots/credentials-captured.png) |
| HASSH fingerprint analysis | [View](screenshots/hassh-fingerprints.png) |
| Cowrie running with JSON logs | [View](screenshots/cowrie-running-logs.png) |
| Attacker inside honeypot shell | [View](screenshots/attacker-inside-honeypot.png) |

## Setup Guide
Full installation instructions: [setup/01-cowrie-install.md](setup/01-cowrie-install.md)

## Planned Improvements
- Real-time Splunk ingestion via Splunk Universal Forwarder
- Expose honeypot to internet for real attacker traffic
- Deploy additional honeypot types (HTTP, FTP)
- Correlate Cowrie logs with Windows Event Logs in Splunk
- Add session duration anomaly detection (very short sessions = automated tools)
