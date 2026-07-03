# Cowrie Honeypot Installation & Configuration

## Environment
- OS: Kali Linux 2025.4
- Cowrie version: Latest (cloned from GitHub)
- Listening port: 2222
- Dedicated user: cowrie

## Prerequisites
Install dependencies as root:

    sudo apt update && sudo apt install -y git python3-virtualenv libssl-dev libffi-dev build-essential python3-dev

## Step 1 — Create Dedicated User
Running Cowrie as root is bad practice. Create a dedicated user:

    sudo adduser --disabled-password cowrie

Press Enter through all fields (Full Name, Room Number etc.) — leave them blank.

## Step 2 — Clone Cowrie Repository

    sudo su - cowrie
    git clone http://github.com/cowrie/cowrie

## Step 3 — Set Up Python Virtual Environment

    cd cowrie
    python3 -m venv cowrie-env
    source cowrie-env/bin/activate
    pip install -r requirements.txt
    pip install -e .

Note: Do NOT cancel pip during installation — interrupting mid-install
breaks the virtual environment. If this happens, rebuild:

    deactivate
    rm -rf cowrie-env
    python3 -m venv cowrie-env
    source cowrie-env/bin/activate

Then re-run pip commands.

## Step 4 — Configure Cowrie

    cp src/cowrie/data/etc/cowrie.cfg.dist etc/cowrie.cfg
    nano etc/cowrie.cfg

Note: The config file is at `src/cowrie/data/etc/cowrie.cfg.dist` —
not `etc/cowrie.cfg.dist`. The etc/ folder is empty by default.

Change hostname to make honeypot look like a real server:

    hostname = webserver01

Save and exit: Ctrl+X → Y → Enter

## Step 5 — Start Cowrie

    cowrie start

Note: Cowrie is installed as a Python package via `pip install -e .`
Use `cowrie start` — NOT `bin/cowrie start`. The bin/ directory
does not contain the start script.

## Step 6 — Verify Cowrie is Listening

    ss -tlnp | grep 2222

Expected output:

    LISTEN 0 50 0.0.0.0:2222 0.0.0.0:* users:(("twistd",pid=XXXXX,fd=11))

## Network Configuration
VMware network adapter must be set to Bridged mode so Cowrie
is accessible from other devices on the same network.

Default NAT mode isolates the VM — attackers on other machines
cannot reach Cowrie on port 2222.

To change: VMware → VM Settings → Network Adapter → Bridged

After switching to Bridged, verify new IP:

    ip a | grep 192.168

Kali will now have a 192.168.1.x IP instead of 192.168.127.x

## Log Location
All attack sessions logged to:

    var/log/cowrie/cowrie.json

## Transferring Logs to Splunk
Start a temporary HTTP server from the cowrie directory:

    python3 -m http.server 8888

Then on the Splunk machine, open browser and navigate to:

    http://192.168.1.72:8888/var/log/cowrie/

Download cowrie.json and upload to Splunk via:
Settings → Add Data → Upload → source type: _json → host: cowrie-honeypot

## Troubleshooting
- If `cowrie start` fails: ensure virtual environment is activated
  (`source cowrie-env/bin/activate`)
- If port 2222 not listening: check cowrie.cfg for port configuration
- If attackers cannot connect: verify VMware is in Bridged mode
- If `pip` not found after venv creation: rebuild venv:
  `rm -rf cowrie-env && python3 -m venv cowrie-env`
- If config file not found: use `find . -name "*.cfg*"` to locate it
