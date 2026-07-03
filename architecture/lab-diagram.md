
# Lab Architecture

## Network Topology

    +------------------+          +------------------+          +------------------+
    |  Second Laptop   |          |   Kali Linux VM  |          |  Main Laptop     |
    |  (Attacker)      | -------> |  (Cowrie         | -------> |  (Splunk SIEM)   |
    |                  |          |   Honeypot)      |          |                  |
    | Windows 11       |          | Port 2222        |          | Splunk 10.4.0    |
    | IP: 192.168.1.83 |          | IP: 192.168.1.72 |          | IP: 192.168.1.234|
    +------------------+          +------------------+          +------------------+

## Component Roles

| Component | Role | IP | OS |
|-----------|------|----|----|
| Second Laptop | Attacker — manual SSH + Hydra brute force | 192.168.1.83 | Windows 11 |
| Kali Linux VM | Cowrie Honeypot (port 2222) + Hydra attacker | 192.168.1.72 | Kali 2025.4 |
| Main Laptop | Splunk SIEM — log ingestion and analysis | 192.168.1.234 | Windows 11 |

## Traffic Flow
1. Attacker (second laptop) connects to Cowrie honeypot on port 2222
2. Cowrie logs all activity silently to cowrie.json
3. Cowrie JSON logs transferred to main laptop
4. Splunk ingests cowrie.json and indexes all events
5. Analyst queries Splunk for IOC extraction and detection

## Honeypot Configuration
- Tool: Cowrie SSH Honeypot
- Listening port: 2222
- Fake hostname: webserver01
- Fake OS presented: Debian GNU/Linux
- Accepts all credentials to lure attackers in
- Logs: var/log/cowrie/cowrie.json

## Limitations
- Manual log transfer to Splunk — not real-time streaming
- Cowrie runs on same machine as attacker VM (Kali)
- No real-time alerting from Cowrie to Splunk
- Lab network only — no exposure to real internet attackers
