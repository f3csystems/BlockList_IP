# Internet Scanner Blacklist

Automatically updated IP blacklist from Internet Scanner alerts (Sekoia.io).

**Last updated:** 2026-07-18 06:00
**Total active IPs:** 2366
**Retention policy:** 30 days — IPs not seen for 30+ days are automatically removed

## Files
- `blacklist.csv` - Full blacklist with metadata (ip, first_seen, last_seen, scan_count, country, scanner_types)
- `blacklist.txt` - Plain text IP list (1 IP per line, for External Dynamic List / Threat Feed)

## Top 10 Scanners
| IP | Scans | Country | Types |
|----|-------|---------|-------|
| 216.180.246.64 | 42 | US | bruteforce, web |
| 34.34.103.195 | 27 | NL | bruteforce, web |
| 209.38.221.33 | 26 | DE | web |
| 51.159.125.208 | 25 | FR | ssh, web |
| 47.130.108.237 | 23 | IN | PaloAlto, bots, email |
| 216.180.246.216 | 22 | US | bots, email, ssh |
| 35.185.62.54 | 21 | US | email, ssh, web |
| 44.247.181.228 | 21 | US | email |
| 52.90.113.213 | 21 | US | ssh, web |
| 5.61.209.224 | 21 | NL | bruteforce, web |

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
