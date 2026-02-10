# Internet Scanner Blacklist

Automatically updated IP blacklist from Internet Scanner alerts.

**Last updated:** 2026-02-10 15:46
**Total IPs:** 211

## Files
- `blacklist.csv` - Full blacklist with metadata (ip, first_seen, last_seen, scan_count, country, scanner_types)
- `blacklist.txt` - Plain text IP list (for firewall import)

## Top 10 Scanners
| IP | Scans | Country | Types |
|----|-------|---------|-------|
| 78.128.112.74 | 51 | BG | bots, bruteforce, cve-2025-55182, ssh |
| 79.124.40.174 | 23 | BG | bots, cve-2025-55182, ssh, web |
| 204.76.203.69 | 12 | NL | adb-abuse, bots, cve-2020-10987, ssh |
| 130.12.180.34 | 10 | GB | bruteforce, web |
| 102.22.20.125 | 9 | GH | N/A |
| 34.158.168.101 | 6 | NL | bots, cve-2025-55182 |
| 64.62.156.80 | 5 | US | email |
| 103.120.189.68 | 5 | IN | cve-2025-55182 |
| 103.20.91.68 | 4 | ID | cve-2025-55182 |
| 185.224.128.16 | 3 | NL | ssh |

## Firewall Import Commands

Use `blacklist.txt` to block all listed IPs. Below are ready-to-use commands for common firewalls.

### FortiGate

**Option 1 — External Threat Feed (recommended, auto-updates):**
```
config system external-resource
    edit "InternetScanner-Blacklist"
        set type address
        set resource "https://raw.githubusercontent.com/avillance/BlockList_IP/main/blacklist.txt"
        set refresh-rate 45
    next
end

config firewall policy
    edit 0
        set name "Block-InternetScanners"
        set srcintf "wan1"
        set dstintf "any"
        set srcaddr "InternetScanner-Blacklist"
        set dstaddr "all"
        set action deny
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
end
```

**Option 2 — One-shot CLI import (from file copied to FortiGate):**
```bash
# Upload blacklist.txt to FortiGate, then:
config firewall address
while read IP; do
    edit "scanner-${IP}"
        set type ipmask
        set subnet ${IP}/32
    next
done < blacklist.txt
end

config firewall addrgrp
    edit "InternetScanner-Blacklist"
        append member $(while read IP; do echo -n "scanner-${IP} "; done < blacklist.txt)
    next
end
```

### Palo Alto (PAN-OS)

**Option 1 — External Dynamic List (EDL, recommended, auto-updates):**

1. Go to **Objects > External Dynamic Lists**
2. Click **Add** and configure:
   - **Name:** `InternetScanner-Blacklist`
   - **Type:** IP List
   - **Source:** `https://raw.githubusercontent.com/avillance/BlockList_IP/main/blacklist.txt`
   - **Repeat:** Every 30 minutes
3. Create a **Security Policy** referencing this EDL as source address with action **Deny**

CLI equivalent:
```
set external-list InternetScanner-Blacklist type ip
set external-list InternetScanner-Blacklist url "https://raw.githubusercontent.com/avillance/BlockList_IP/main/blacklist.txt"
set external-list InternetScanner-Blacklist recurring five-minute

set rulebase security rules Block-InternetScanners from any to any
set rulebase security rules Block-InternetScanners source InternetScanner-Blacklist
set rulebase security rules Block-InternetScanners action deny
set rulebase security rules Block-InternetScanners log-start yes
```

**Option 2 — Batch CLI import:**
```bash
# From a management host with API access:
while read IP; do
    curl -k -X POST "https://<FIREWALL>/api/?type=config" \
        -d "key=<API_KEY>" \
        -d "action=set" \
        -d "xpath=/config/devices/entry/vsys/entry/address/entry[@name='scanner-${IP}']" \
        -d "element=<ip-netmask>${IP}/32</ip-netmask>"
done < blacklist.txt
```

### Check Point

**Option 1 — Updatable Object / External Feed (recommended, R80.10+):**

1. In **SmartConsole**, go to **New > More > Network Feed**
2. Configure:
   - **Name:** `InternetScanner-Blacklist`
   - **URL:** `https://raw.githubusercontent.com/avillance/BlockList_IP/main/blacklist.txt`
   - **Update interval:** 30 minutes
   - **Content type:** IP Address
3. Use this object as **Source** in a **Drop** rule
4. **Install Policy**

**Option 2 — mgmt_cli batch import:**
```bash
# From Check Point management server:
mgmt_cli login user <USER> password <PASS> > sid.txt
SID=$(cat sid.txt | jq -r '.sid')

while read IP; do
    mgmt_cli add host name "scanner-${IP}" ip-address "${IP}" -s "$SID"
done < blacklist.txt

mgmt_cli add group name "InternetScanner-Blacklist" -s "$SID"
while read IP; do
    mgmt_cli set group name "InternetScanner-Blacklist" members.add "scanner-${IP}" -s "$SID"
done < blacklist.txt

mgmt_cli publish -s "$SID"
mgmt_cli logout -s "$SID"
```

> **Note:** For all firewalls, **Option 1 (External Feed/EDL)** is strongly recommended.
> The firewall fetches `blacklist.txt` directly from this repository at regular intervals,
> so new scanner IPs are blocked automatically without manual intervention.

---
*Updated automatically every 45 minutes*
