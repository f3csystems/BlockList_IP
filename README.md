# Internet Scanner Blacklist

Automatically updated IP blacklist from Internet Scanner alerts (Sekoia.io).

**Last updated:** 2026-02-11 09:39
**Total active IPs:** 398
**Retention policy:** 30 days — IPs not seen for 30+ days are automatically removed

## Files
- `blacklist.csv` - Full blacklist with metadata (ip, first_seen, last_seen, scan_count, country, scanner_types)
- `blacklist.txt` - Plain text IP list (1 IP per line, for External Dynamic List / Threat Feed)

## Top 10 Scanners
| IP | Scans | Country | Types |
|----|-------|---------|-------|
| 78.128.112.74 | 134 | BG | PaloAlto, bots, bruteforce, cve-2023-1389-2, cve-2025-55182, dicom, email, onyphe, ssh, web, yarn |
| 185.224.128.16 | 35 | NL | PaloAlto, adb-abuse, bots, ssh |
| 79.124.40.174 | 30 | BG | bots, cve-2025-55182, ssh, web |
| 66.240.236.116 | 27 | US | PaloAlto, bruteforce, cve-2020-10987, cve-2025-55182, email, ssh |
| 130.12.180.34 | 26 | GB | bots, bruteforce, ssh, web |
| 204.76.203.69 | 19 | NL | adb-abuse, bots, cve-2020-10987, ssh |
| 34.158.168.101 | 15 | NL | bots, cve-2025-55182, ssh |
| 102.22.20.125 | 9 | GH | nan |
| 103.120.189.68 | 9 | IN | cve-2025-55182, ssh |
| 103.20.91.68 | 8 | ID | PaloAlto, cve-2025-55182, ssh |

## Firewall Integration — External Dynamic Lists / Threat Feeds

> **Important:** This blacklist **must** be consumed via External Dynamic Lists (EDL) or Threat Feeds.
> Do **not** import the IPs manually or via script — only dynamic feeds ensure automatic updates
> and respect the 30-day retention policy (expired IPs are automatically removed).

The file `blacklist.txt` contains one IP per line and is updated every 45 minutes.
IPs not seen for 30+ days are automatically purged to keep the list relevant.

### FortiGate — External Threat Feed

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

The FortiGate will automatically fetch and refresh the IP list every 45 minutes.

### Palo Alto — External Dynamic List (EDL)

**GUI:**

1. Go to **Objects > External Dynamic Lists**
2. Click **Add** and configure:
   - **Name:** `InternetScanner-Blacklist`
   - **Type:** IP List
   - **Source:** `https://raw.githubusercontent.com/avillance/BlockList_IP/main/blacklist.txt`
   - **Repeat:** Every 30 minutes
3. Create a **Security Policy** referencing this EDL as source address with action **Deny**

**CLI equivalent:**
```
set external-list InternetScanner-Blacklist type ip
set external-list InternetScanner-Blacklist url "https://raw.githubusercontent.com/avillance/BlockList_IP/main/blacklist.txt"
set external-list InternetScanner-Blacklist recurring five-minute

set rulebase security rules Block-InternetScanners from any to any
set rulebase security rules Block-InternetScanners source InternetScanner-Blacklist
set rulebase security rules Block-InternetScanners action deny
set rulebase security rules Block-InternetScanners log-start yes
```

### Check Point — Network Feed (R80.10+)

1. In **SmartConsole**, go to **New > More > Network Feed**
2. Configure:
   - **Name:** `InternetScanner-Blacklist`
   - **URL:** `https://raw.githubusercontent.com/avillance/BlockList_IP/main/blacklist.txt`
   - **Update interval:** 30 minutes
   - **Content type:** IP Address
3. Use this object as **Source** in a **Drop** rule
4. **Install Policy**

---
*Updated automatically every 45 minutes — IPs expire after 30 days without activity*
