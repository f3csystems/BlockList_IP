# Internet Scanner Blacklist

Automatically updated blacklist of IP addresses observed performing internet-wide scanning.

**Last updated:** 2026-10-10 02:35
**Total active IPs:** 2393
**Retention policy:** 30 days — IPs not seen for 30+ days are automatically removed

## Files
- `blacklist.csv` - Full blacklist with metadata (ip, first_seen, last_seen, scan_count, country, scanner_types)
- `blacklist.txt` - Plain text IP list (1 IP per line, for External Dynamic List / Threat Feed)

## Top 10 Scanners
| IP | Scans | Country | Types |
|----|-------|---------|-------|
| 147.185.132.165 | 12971 | US | PaloAlto, email |
| 205.210.31.222 | 11386 | BR | PaloAlto |
| 198.235.24.183 | 11011 | BE | PaloAlto |
| 198.235.24.99 | 10920 | TW | PaloAlto, ssh |
| 147.185.132.21 | 10473 | US | Censys, PaloAlto |
| 147.185.132.207 | 9945 | US | PaloAlto |
| 147.185.132.81 | 9907 | US | PaloAlto |
| 147.185.132.45 | 8623 | US | PaloAlto |
| 198.235.24.104 | 8604 | TW | PaloAlto |
| 205.210.31.85 | 8338 | US | PaloAlto, bruteforce, ssh |

## Firewall Integration — External Dynamic Lists / Threat Feeds

> **Important:** This blacklist **must** be consumed via External Dynamic Lists (EDL) or Threat Feeds.
> Do **not** import the IPs manually or via script — only dynamic feeds ensure automatic updates
> and respect the 30-day retention policy (expired IPs are automatically removed).

The file `blacklist.txt` contains one IP per line and is updated every 30 minutes.
IPs not seen for 30+ days are automatically purged to keep the list relevant.

### FortiGate — External Threat Feed

```
config system external-resource
    edit "InternetScanner-Blacklist"
        set type address
        set resource "https://raw.githubusercontent.com/f3csystems/BlockList_IP/main/blacklist.txt"
        set refresh-rate 30
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

The FortiGate will automatically fetch and refresh the IP list every 30 minutes.

### Palo Alto — External Dynamic List (EDL)

**GUI:**

1. Go to **Objects > External Dynamic Lists**
2. Click **Add** and configure:
   - **Name:** `InternetScanner-Blacklist`
   - **Type:** IP List
   - **Source:** `https://raw.githubusercontent.com/f3csystems/BlockList_IP/main/blacklist.txt`
   - **Repeat:** Every 30 minutes
3. Create a **Security Policy** referencing this EDL as source address with action **Deny**

**CLI equivalent:**
```
set external-list InternetScanner-Blacklist type ip
set external-list InternetScanner-Blacklist url "https://raw.githubusercontent.com/f3csystems/BlockList_IP/main/blacklist.txt"
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
   - **URL:** `https://raw.githubusercontent.com/f3csystems/BlockList_IP/main/blacklist.txt`
   - **Update interval:** 30 minutes
   - **Content type:** IP Address
3. Use this object as **Source** in a **Drop** rule
4. **Install Policy**

---
*Updated automatically every 30 minutes — IPs expire after 30 days without activity*
