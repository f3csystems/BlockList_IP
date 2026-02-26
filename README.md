# Internet Scanner Blacklist

Automatically updated IP blacklist from Internet Scanner alerts (Sekoia.io).

**Last updated:** 2026-02-26 06:45
**Total active IPs:** 2389
**Retention policy:** 30 days — IPs not seen for 30+ days are automatically removed

## Files
- `blacklist.csv` - Full blacklist with metadata (ip, first_seen, last_seen, scan_count, country, scanner_types)
- `blacklist.txt` - Plain text IP list (1 IP per line, for External Dynamic List / Threat Feed)

## Top 10 Scanners
| IP | Scans | Country | Types |
|----|-------|---------|-------|
| 78.128.112.74 | 451 | BG | PaloAlto, Rapid7, adb-abuse, bots, bruteforce, cve-2020-10987, cve-2023-1389-2, cve-2025-55182, dicom, email, ftp, onyphe, rdp, ssh, web, yarn |
| 16.58.56.214 | 49 | US | PaloAlto, bots, bruteforce, cve-2025-55182, ssh, web |
| 176.65.139.30 | 45 | DE | PaloAlto, adb-abuse, bots, bruteforce, cve-2025-55182, ssh, web, yarn |
| 45.135.193.11 | 39 | DE | PaloAlto, bots, bruteforce, cve-2020-10987, ssh, web |
| 185.224.128.16 | 39 | NL | PaloAlto, adb-abuse, bots, ssh |
| 87.120.191.67 | 33 | NL | PaloAlto, adb-abuse, bots, bruteforce, cve-2025-55182-2, onyphe, web |
| 79.124.40.174 | 32 | BG | Rapid7, bots, cve-2025-55182, ssh, web |
| 45.148.10.124 | 31 | NL | adb-abuse, bruteforce, cve-2025-55182, cve-2025-55182-2, ssh, web |
| 130.12.180.55 | 30 | DE | PaloAlto, bruteforce, cve-2025-55182, ssh |
| 34.158.168.101 | 29 | NL | PaloAlto, adb-abuse, bots, bruteforce, cve-2025-55182, ftp, onyphe, ssh, web |

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
        set resource "https://raw.githubusercontent.com/f3cSystems/BlockList_IP/main/blacklist.txt"
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
   - **Source:** `https://raw.githubusercontent.com/f3cSystems/BlockList_IP/main/blacklist.txt`
   - **Repeat:** Every 30 minutes
3. Create a **Security Policy** referencing this EDL as source address with action **Deny**

**CLI equivalent:**
```
set external-list InternetScanner-Blacklist type ip
set external-list InternetScanner-Blacklist url "https://raw.githubusercontent.com/f3cSystems/BlockList_IP/main/blacklist.txt"
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
   - **URL:** `https://raw.githubusercontent.com/f3cSystems/BlockList_IP/main/blacklist.txt`
   - **Update interval:** 30 minutes
   - **Content type:** IP Address
3. Use this object as **Source** in a **Drop** rule
4. **Install Policy**

---
*Updated automatically every 45 minutes — IPs expire after 30 days without activity*
