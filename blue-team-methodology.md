# Blue Team Methodology

## Method Map

- `1` **Incident Response & Triage**
  - `1.1` **Windows (CMD)**
    - `1.1.1` System information
    - `1.1.2` User and service enumeration
    - `1.1.3` Network connections
  - `1.2` **Windows (PowerShell)**
    - `1.2.1` System and network enumeration
    - `1.2.2` User and service enumeration
    - `1.2.3` Scheduled tasks
    - `1.2.4` Zone.Identifier
  - `1.3` **DeepBlue CLI**
  - `1.4` **Linux**
    - `1.4.1` File metadata inspection
    - `1.4.2` Log files → see §5
- `2` **Threat Intelligence & IOC Lookup**
  - `2.1` **IOC Types**
  - `2.2` **Blocking Methods**
  - `2.3` **Threat Intelligence Sites**
- `3` **Email & Phishing Analysis**
  - `3.1` **Phishing Tools**
  - `3.2` **Phishing Analysis Sites**
- `4` **Digital Forensics — Windows**
  - `4.1` **Registry**
    - `4.1.1` OS information
    - `4.1.2` Computer name
    - `4.1.3` USB device history
    - `4.1.4` User persistence (Run key)
  - `4.2` **Event Logs**
    - `4.2.1` Logon events
    - `4.2.2` Log file locations
  - `4.3` **File System Artifacts**
    - `4.3.1` LNK files
    - `4.3.2` Prefetch
    - `4.3.3` Jump Lists
    - `4.3.4` Recycle Bin
    - `4.3.5` WPDNSE
  - `4.4` **Autopsy**
  - `4.5` **Process Memory Dumping**
- `5` **Digital Forensics — Linux**
  - `5.1` **Key File Locations**
  - `5.2` **Log Files**
  - `5.3` **File Carving**
  - `5.4` **Steganography**
- `6` **Digital Forensics — Android**
  - `6.1` **Device Identification**
  - `6.2` **Connectivity**
  - `6.3` **System State**
- `7` **Memory Forensics**
  - `7.1` **Volatility 2**
  - `7.2` **Volatility 3**
- `8` **Log Analysis & SIEM**
  - `8.1` **Splunk**
  - `8.2` **Sysmon & Event Logs**
- `9` **Network Analysis**
  - `9.1` **Wireshark**
  - `9.2` **Network Miner**
- `10` **Malware Analysis**
  - `10.1` **Sandboxing Sites**
  - `10.2` **Analysis Tools**
- `11` **Useful Sites**

## 1. Incident Response & Triage

### 1.1 Windows (CMD)

#### 1.1.1 System information <sup>· BTL1</sup>

```cmd
ipconfig /all
```

<details>
<summary>Details</summary>

**Description**

- Displays full TCP/IP configuration for all network adapters.
- Includes IP address, MAC address, DNS servers, DHCP lease info, and adapter state.

</details>

#### 1.1.2 User and service enumeration <sup>· BTL1</sup>

**List running processes:**

```cmd
tasklist
```

**List process details:**

```cmd
wmic process get description,executablepath
```

**List local users and groups:**

```cmd
net user
net localgroup
```

**List services:**

```cmd
sc query | more
```

<details>
<summary>Details</summary>

**Description**

- `tasklist` — Quick process listing by name and PID.
- `wmic process get description,executablepath` — Includes the full executable path for each running process.
- `net user` — Lists all local user accounts on the system.
- `net localgroup` — Lists all local groups and their members.
- `sc query` — Enumerates all installed Windows services and their current state.

</details>

#### 1.1.3 Network connections <sup>· BTL1</sup>

```cmd
netstat -ab
```

<details>
<summary>Details</summary>

**Description**

- Lists all active TCP/UDP connections and listening ports.
- `-a` — Displays all connections and listening ports.
- `-b` — Shows the executable involved in each connection (requires elevation).

</details>

### 1.2 Windows (PowerShell)

#### 1.2.1 System and network enumeration <sup>· BTL1</sup>

```powershell
Get-NetIPConfiguration
Get-NetIPAddress
```

<details>
<summary>Details</summary>

**Description**

- `Get-NetIPConfiguration` — Displays IP configuration per network adapter (IP, gateway, DNS).
- `Get-NetIPAddress` — Lists all IP addresses configured on the system, including non-primary and link-local addresses.

</details>

#### 1.2.2 User and service enumeration <sup>· BTL1</sup>

**List local users:**

```powershell
Get-LocalUser
```

**List running services:**

```powershell
Get-Service | Where Status -eq "Running"
```

**List processes with detailed view:**

```powershell
Get-Process | Format-Table -View priority
```

<details>
<summary>Details</summary>

**Description**

- `Get-LocalUser` — Enumerates all local user accounts, including disabled and built-in accounts.
- `Get-Service | Where Status -eq "Running"` — Filters for currently running services only.
- `Get-Process | Format-Table -View priority` — Displays processes with a focus on priority and resource usage.

</details>

#### 1.2.3 Scheduled tasks <sup>· BTL1</sup>

```powershell
Get-ScheduledTask
```

<details>
<summary>Details</summary>

**Description**

- Lists all scheduled tasks on the system; useful for detecting persistence mechanisms.
- See [4.1.4](#414-user-persistence-run-key) for Registry-based persistence analysis.

</details>

#### 1.2.4 Zone.Identifier <sup>· BTL1</sup>

```powershell
Get-Content -Path <FILE> -Stream zone.identifier
Get-Content -Path <FILE> -Stream *
```

<details>
<summary>Details</summary>

**Description**

- Windows attaches a Zone.Identifier NTFS alternate data stream to files downloaded from the internet. Contains the source URL and referrer.
- A missing stream does not guarantee the file wasn't downloaded — it may have been stripped.
- See [4.3.5](#435-wpdnse) for WPDNSE, a related malware indicator.

</details>

### 1.3 DeepBlue CLI <sup>· BTL1</sup>

```powershell
Set-ExecutionPolicy Bypass -Scope CurrentUser
.\DeepBlue.ps1 -log security
.\DeepBlue.ps1 -log system
```

<details>
<summary>Details</summary>

**Description**

- DeepBlue CLI parses Windows Event Logs for common attack patterns.
- Detects Metasploit-style activity, suspicious service creation (Event ID 7045), and other attack indicators.

**Parameters**

- `-log security` — Analyses the Security event log.
- `-log system` — Analyses the System event log.

</details>

### 1.4 Linux

#### 1.4.1 File metadata inspection <sup>· BTL1</sup>

```bash
ls -lisap
stat <FILE>
exiftool <FILE>
chown <USER>:<GROUP> <FILE>
```

<details>
<summary>Details</summary>

**Description**

- `ls -lisap` — Detailed file listing including inode numbers, sizes, permissions, and hidden files.
- `stat` — Displays detailed file metadata (access/modify/change timestamps, inode, permissions, ownership).
- `exiftool` — Extracts comprehensive metadata from files (images, documents, executables). See [5.4](#54-steganography) for steganography-specific usage.
- `chown` — Changes file ownership; useful when restoring permissions on extracted artifacts.

</details>

#### 1.4.2 Log files

For log file locations and descriptions, see [5.2 Log Files](#52-log-files).

For key file paths and their contents, see [5.1 Key File Locations](#51-key-file-locations).

## 2. Threat Intelligence & IOC Lookup

### 2.1 IOC Types <sup>· BTL1</sup>

| Category | Examples |
|---|---|
| **Email** | Sender address, subject line, attachments, embedded URLs |
| **IP Address** | Command-and-control servers, scanning sources, phishing hosts |
| **Domain / URL** | Phishing pages, malware delivery sites, redirectors |
| **File Hash / Name** | MD5, SHA-1, SHA-256 hashes; malicious file names |

### 2.2 Blocking Methods <sup>· BTL1</sup>

| Method | Targets |
|---|---|
| **Email** | Sender address, domain, server IP, subject line |
| **Web** | URL, domain, IP address of malicious site |
| **File** | Hash (MD5/SHA256), file name pattern |

<details>
<summary>Details</summary>

**Description**

- Email blocking can be applied at the gateway level based on sender, domain, SMTP server IP, or keywords in the subject line.
- Web blocking prevents users from accessing known-malicious URLs, domains, or IPs via proxy or DNS filtering.
- File blocking uses application whitelisting or AV signatures to prevent known-malicious files from executing.

</details>

### 2.3 Threat Intelligence Sites

| Site | Purpose |
|---|---|
| [VirusTotal](https://www.virustotal.com) | Multi-engine file, URL, domain, and IP analysis |
| [Talos Intelligence](https://www.talosintelligence.com/) | Cisco's reputation centre for IPs, domains, files |
| [URLhaus](https://urlhaus.abuse.ch) | Tracks malicious URLs and payloads |
| [MalwareBazaar](https://bazaar.abuse.ch/) | Malware sample database and IOC sharing |
| [PhishTank](https://www.phishtank.org/) | Community-verified phishing URL database |
| [Joe Sandbox](https://www.joesandbox.com/) | Automated malware analysis sandbox |
| [Hybrid Analysis](https://www.hybrid-analysis.com) | CrowdStrike's online sandbox with detailed reports |

## 3. Email & Phishing Analysis

### 3.1 Phishing Tools

| Tool | Purpose |
|---|---|
| **PhishTool** | Comprehensive phishing email analysis — headers, attachments, body |
| **OneNote Analyzer** | [GitHub](https://github.com/knight0x07/OneNoteAnalyzer) — Parses `.one` files for embedded scripts, URLs, and suspicious content |

### 3.2 Phishing Analysis Sites

| Site | Purpose |
|---|---|
| [PhishTank](https://www.phishtank.org/) | Verify whether a URL is a known phishing page |
| [Domaintools Whois](http://whois.domaintools.com) | Domain registration details, ownership history, creation date |
| [WannaBrowser](https://www.wannabrowser.net) | Renders a URL and shows how it appears in a browser |
| [URL2PNG](https://www.url2png.com) | Generates a screenshot of a webpage for safe viewing |

## 4. Digital Forensics — Windows

### 4.1 Registry

#### 4.1.1 OS information <sup>· BTL1</sup>

**Key:** `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion`

| Value | Description |
|---|---|
| `ProductName` | Windows edition (e.g. Windows 10 Pro) |
| `CurrentBuild` | OS build number |
| `RegisterOwner` | Registered owner name |
| `InstallDate` | OS installation timestamp (Unix epoch) |

**Registry hive disk location:** `C:\Windows\System32\config`

<details>
<summary>Details</summary>

**Reference:** [Microsoft — Registry Hives](https://learn.microsoft.com/en-us/windows/win32/sysinfo/registry-hives)

</details>

#### 4.1.2 Computer name <sup>· BTL1</sup>

**Volatile (runtime only):**

```
HKLM\SYSTEM\CurrentControlSet\Control\ComputerName\ActiveComputerName
```

**Non-volatile (persists across reboots):**

```
HKLM\SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName
```

<details>
<summary>Details</summary>

**Description**

- The volatile key only reflects the name while the system is running and is lost after reboot.
- The non-volatile key stores the permanent computer name that persists.

</details>

#### 4.1.3 USB device history <sup>· BTL1</sup>

```
HKLM\SYSTEM\CurrentControlSet\Enum\USBSTOR
```

<details>
<summary>Details</summary>

**Description**

- Records every USB storage device ever connected to the system.
- Each subkey contains the device's Vendor ID (VID), Product ID (PID), and serial number.
- Timestamps in the subkey metadata indicate first and last connection times.

</details>

#### 4.1.4 User persistence (Run key) <sup>· BTL1</sup>

```
HKU\<User_SID>\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\<Program_Name>
```

<details>
<summary>Details</summary>

**Description**

- Programs listed under this key automatically execute when the specific user logs on.
- A common persistence mechanism used by both legitimate software and malware.
- Replace `<User_SID>` with the user's Security Identifier (e.g. `S-1-5-21-4001622725-2027095096-2530479061-1117`).

</details>

### 4.2 Event Logs

#### 4.2.1 Logon events <sup>· BTL1</sup>

| Event ID | Description |
|---|---|
| **4624** | Successful logon |
| **4625** | Failed logon |
| **4634** | Logoff |
| **4672** | Special privileges assigned to new logon (admin logon) |

**Logon types (Event 4624):**

| Type | Description |
|---|---|
| 2 | Interactive (local console) |
| 3 | Network (SMB share, remote access) |
| 4 | Batch (scheduled task) |
| 5 | Service |
| 7 | Unlock (workstation unlock) |
| 8 | Network cleartext (IIS basic auth) |
| 9 | New credentials (RunAs, secondary logon) |
| 10 | Remote interactive (RDP) |
| 11 | Cached interactive (cached domain credentials) |

<details>
<summary>Details</summary>

**Description**

- Logon type 10 (RDP) and type 3 (Network) are frequently abused by attackers for lateral movement.
- Repeated 4625 events from a single source IP indicate brute-force or password spraying attempts.
- Event 4672 always accompanies an administrative logon via 4624.

</details>

#### 4.2.2 Log file locations <sup>· BTL1</sup>

**Legacy (Windows XP / Server 2003):**

```
%WinDir%\system32\Config\*.evt
```

**Modern (Windows Vista+ / Server 2008+):**

```
%WinDir%\system32\WinEVT\Logs\*.evtx
```

<details>
<summary>Details</summary>

**Description**

- Legacy `.evt` format is binary and not directly readable outside the Event Viewer.
- Modern `.evtx` files are in XML-based format, parseable with PowerShell's `Get-WinEvent` or tools like LogParser.
- Key log files: `Security.evtx` (logon/access events), `System.evtx` (service/driver events), `Application.evtx`.

**Reference:** [Ultimate Windows Security Log Encyclopedia](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/default.aspx)

</details>

### 4.3 File System Artifacts

#### 4.3.1 LNK files <sup>· BTL1</sup>

```
C:\Users\<USERNAME>\AppData\Roaming\Microsoft\Windows\Recent
```

<details>
<summary>Details</summary>

**Description**

- LNK (shortcut) files are created whenever a user opens a file, local or remote.
- Each .lnk file contains: original file path, file size, MAC timestamps of the target file, and volume serial number.
- Even if the original file is deleted, the LNK file persists and provides forensic evidence of file access.

</details>

#### 4.3.2 Prefetch <sup>· BTL1</sup>

```
C:\Windows\Prefetch
PECmd.exe -f <PREFETCH_FILE>
PECmd.exe -d C:\Windows\Prefetch -k rows
```

<details>
<summary>Details</summary>

**Description**

- Windows Prefetch stores execution metadata to speed up application launches.
- Each `.pf` file contains: application name, execution count, last run timestamp, and referenced file/dll paths.
- `PECmd.exe` (Eric Zimmerman's tool) parses Prefetch files into human-readable output.

**Parameters**

- `-f <PREFETCH_FILE>` — Parse a single Prefetch file.
- `-d <DIRECTORY>` — Parse all Prefetch files in a directory.
- `-k rows` — Output format: rows.

</details>

#### 4.3.3 Jump Lists <sup>· BTL1</sup>

```
C:\Users\<USERNAME>\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations
C:\Users\<USERNAME>\AppData\Roaming\Microsoft\Windows\Recent\CustomDestinations
```

<details>
<summary>Details</summary>

**Description**

- Jump Lists track recently opened files per application (e.g. Paint, Word, Notepad).
- `AutomaticDestinations` — Created automatically by the OS with a unique AppID per application.
- `CustomDestinations` — Created when a user explicitly pins a file to the taskbar or Start menu.
- Files are in OLE Compound format; parse with JLECmd or JumpList Explorer.

</details>

#### 4.3.4 Recycle Bin <sup>· BTL1</sup>

```
C:\$Recycle.Bin
wmic useraccount get name,SID
get-childitem -force C:\$Recycle.Bin
RBCmd.exe -d C:\$Recycle.Bin
```

<details>
<summary>Details</summary>

**Description**

- The Recycle Bin contains files deleted by the user with their original metadata preserved.
- `$R` files — The original deleted file content.
- `$I` files — Metadata (original file path, deletion timestamp, file size).
- Match the user SID to the folder under `C:\$Recycle.Bin` to attribute deleted files to a specific user.

**Parameters**

- `wmic useraccount get name,SID` — Maps usernames to their Security Identifiers.
- `get-childitem -force` — Lists hidden items in the Recycle Bin.
- `RBCmd.exe -d <DIRECTORY>` — Parses Recycle Bin metadata with Eric Zimmerman's RBCmd.

**Reference:** [df-stream.com — Recycle Bin $I Files](https://df-stream.com/2016/04/fun-with-recycle-bin-i-files-windows-10/)

</details>

#### 4.3.5 WPDNSE <sup>· BTL1</sup>

<details>
<summary>Details</summary>

**Description**

- Windows Portable Device Namespace Extension. If found under `AppData\Roaming` instead of the expected `Temp` directory, it may indicate malware activity related to mobile device enumeration.
- Check Zone.Identifier alternate data streams on suspicious files with the commands in [1.2.4](#124-zoneidentifier).

</details>

### 4.4 Autopsy <sup>· BTL1</sup>

| Application | Forensic Artifact | Location |
|---|---|---|
| **Signal Messenger** | `config.json`, `db.sqlite` | Application data directory |
| **Tor Browser** | `places.sqlite` (browsing history, bookmarks) | Tor Browser profile directory |
| **Sticky Notes** | `plum.sqlite` | Application data directory |
| **Windows Prefetch** | `.pf` files | `/Windows/Prefetch` |
| **Thumbcache** | Thumbnail database files | `/Explorer/` |

<details>
<summary>Details</summary>

**Description**

- Autopsy is an open-source digital forensics platform for disk image analysis.
- Key artifacts to look for include messaging app databases (Signal), browsing history (Tor), system usage (Prefetch), and cached images (Thumbcache).

</details>

### 4.5 Process Memory Dumping <sup>· BTL1</sup>

```powershell
Get-Process | findstr -I "<PROCESS>"
procdump.exe -ma <PID>
```

<details>
<summary>Details</summary>

**Description**

- `procdump` (Sysinternals) creates a memory dump of a running process for offline analysis with Volatility (see [7. Memory Forensics](#7-memory-forensics)) or Mimikatz.
- `findstr -I` performs a case-insensitive search for the target process name before dumping.

**Parameters**

- `-ma` — Full memory dump.
- `<PID>` — Process ID obtained from `Get-Process` or `tasklist`.

</details>

## 5. Digital Forensics — Linux

### 5.1 Key File Locations <sup>· BTL1</sup>

| File | Content |
|---|---|
| `/etc/passwd` | Local user accounts (no passwords on modern systems) |
| `/etc/shadow` | Hashed passwords (requires root to read) |
| `/var/lib/dpkg/status` | Installed package list and versions (Debian/Ubuntu) |

<details>
<summary>Details</summary>

**Description**

- `/etc/shadow` — Contains the actual password hashes. If readable by non-root, the system is misconfigured.
- `/var/lib/dpkg/status` — Equivalent to `rpm -qa` on RHEL systems; reveals installed software for vulnerability assessment.
- Shell command history is covered under [5.2 Log Files](#52-log-files).

</details>

### 5.2 Log Files <sup>· BTL1</sup>

| Log | Purpose |
|---|---|
| `/var/log/auth.log` | Authentication events (Debian/Ubuntu) |
| `/var/log/secure` | Authentication events (RHEL/CentOS) |
| `/var/log/dpkg.log` | Package installation/removal |
| `/var/log/btmp` | Failed login attempts (binary; use `lastb`) |
| `/var/log/cron` | Cron job execution |
| `/var/log/faillog` | Failed login records (binary; use `faillog`) |
| `/var/log/apache2/access.log` | Apache HTTP access log |
| `~/.bash_history` | Shell command history per user |

<details>
<summary>Details</summary>

**Description**

- `lastb` reads `/var/log/btmp` and shows failed login attempts with source IPs.
- `faillog` reads `/var/log/faillog` and displays per-user failure counts and timestamps.
- Apache access logs may contain evidence of web-based attacks (SQL injection, path traversal, command injection).
- `.bash_history` — May be deleted or symlinked to `/dev/null` by attackers to hide activity.

</details>

### 5.3 File Carving <sup>· BTL1</sup>

```bash
scalpel -o <OUTPUT_DIR> <DISK_IMAGE>
```

**Configuration file:** `/etc/scalpel/scalpel.conf`

<details>
<summary>Details</summary>

**Description**

- Scalpel carves files from a raw disk image based on file headers and footers.
- Edit `/etc/scalpel/scalpel.conf` to uncomment the file types you want to carve (e.g. jpg, pdf, zip).
- Carving recovers files even after deletion, as long as the data blocks haven't been overwritten.

**Parameters**

- `-o <OUTPUT_DIR>` — Directory where carved files are written.
- `<DISK_IMAGE>` — Path to the raw disk image or partition.

</details>

### 5.4 Steganography <sup>· BTL1</sup>

**Embed a message:**

```bash
steghide embed -cf <COVER_FILE> -ef <EMBED_FILE>
```

**Extract a hidden message:**

```bash
steghide extract -sf <STEGO_FILE>
```

**Embed data in image metadata:**

```bash
exiftool -Comment="<HIDDEN_MESSAGE>" <IMAGE_FILE>
```

<details>
<summary>Details</summary>

**Description**

- Steganography hides data within innocuous files (images, audio, video).
- `steghide` embeds arbitrary files inside JPEG, BMP, WAV, and AU files with optional passphrase encryption.
- `exiftool -Comment=` writes arbitrary text into an image's EXIF Comment field.

**Parameters**

- `-cf` — Cover file (the carrier).
- `-ef` — File to embed (the secret).
- `-sf` — Stego file (file with hidden data to extract).
- `-Comment="..."` — EXIF comment to write or overwrite.

</details>

## 6. Digital Forensics — Android

### 6.1 Device Identification

#### IMEI

```
/data/drm/pvt/ahrh
```

**Reference:** [IMEI.info Lookup](https://www.imei.info/)

<details>
<summary>Details</summary>

**Description**

- International Mobile Equipment Identity — unique 15-digit identifier for the device hardware.
- IMEI breakdown: TAC (digits 1–6), Final Assembly Code (7–8), Serial Number (9–14), Check Digit (15).

</details>

#### SIM Card (ICCID / IMSI)

```
/data/user_de/<USER_NUMBER>/com.android.providers.telephony/databases/telephony.db
```

<details>
<summary>Details</summary>

**Description**

- **ICCID** (Integrated Circuit Card Identifier) — 19–20 digit unique SIM card identifier. Includes issuing country (MCC), network (MNC), and serial number.
- **IMSI** (International Mobile Subscriber Identity) — 15-digit subscriber identifier stored on the SIM. MCC (1–3), MNC (4–6), MSIN (7–15).

</details>

#### Android ID & Bluetooth

```
/data/system/users/<USER_NUMBER>/settings_secure.xml
```

<details>
<summary>Details</summary>

**Description**

- Contains the unique Android ID, Bluetooth adapter name, and Bluetooth MAC address for the device.

</details>

#### Android OS Version

```
/data/system/usagestats/<USER_NUMBER>/version
```

### 6.2 Connectivity

#### Wi-Fi Networks

```
/data/misc/wifi/WifiConfigStore.xml
```

<details>
<summary>Details</summary>

**Description**

- Stores all Wi-Fi networks the device has ever connected to, including SSIDs, security types, and pre-shared keys (if saved).

</details>

#### Wi-Fi Mobile Hotspot

```
/data/misc/wifi/softap.conf
```

<details>
<summary>Details</summary>

**Description**

- Contains hotspot settings: SSID, security mode (WPA2/WPA3), password, channel, and band (2.4/5 GHz).

</details>

#### Bluetooth Paired Devices

```
/data/misc/bluedroid/bt_config.conf
/data/misc/bluedroid/bt_config.bak
```

<details>
<summary>Details</summary>

**Description**

- `bt_config.conf` — Lists all paired Bluetooth devices with names, MAC addresses, pairing timestamps, connection history, profiles (A2DP, HFP), and security keys (link keys, PINs).
- `bt_config.bak` — Backup of the Bluetooth configuration.

</details>

#### Bluetooth Battery Usage

```
/data/user/<USER_NUMBER>/com.google.android.apps.turbo/databases/bluetooth.db
```

#### ADB Hosts

```
/data/misc/adb/adb_keys
```

<details>
<summary>Details</summary>

**Description**

- ADB (Android Debug Bridge) enables USB or TCP debugging connections. This file lists the public keys of computers authorised for ADB access.
- Unauthorised ADB keys may indicate an attacker who enabled debugging for remote access.

</details>

### 6.3 System State

#### Shutdown Checkpoints

```
/data/system/shutdown-checkpoints/checkpoints-<NUMBER>
```

**Reasons:** `USER_SHUTDOWN`, `LOW_BATTERY`, `THERMAL_SHUTDOWN`, `SOFTWARE_UPDATE`, `CRASH`

#### Last Boot Time

```
/data/misc/bootstat/last_boot_time_utc
```

#### Last Factory Reset

```
/data/misc/bootstat/factory_reset
```

#### Lock Screen Settings

```
/data/system/locksettings.db
```

| `lockscreen.password_type` | Meaning |
|---|---|
| 0 | None |
| 65536 | Pattern |
| 131072 | PIN |
| 196608 | Password |
| 262144 | Biometric |
| 327680 | Face |
| 393216 | Iris |

#### Battery Percentage

```
/data/user/<USER_NUMBER>/com.google.android.apps.turbo/databases/turbo.db
```

## 7. Memory Forensics

### 7.1 Volatility 2 <sup>· BTL1</sup>

```bash
vol.py -f <MEMORY_DUMP> imageinfo
vol.py -f <MEMORY_DUMP> pslist
vol.py -f <MEMORY_DUMP> pstree
vol.py -f <MEMORY_DUMP> psscan
vol.py -f <MEMORY_DUMP> psxview
vol.py -f <MEMORY_DUMP> netscan
vol.py -f <MEMORY_DUMP> connections
vol.py -f <MEMORY_DUMP> connscan
vol.py -f <MEMORY_DUMP> cmdline
vol.py -f <MEMORY_DUMP> filescan
vol.py -f <MEMORY_DUMP> dumpfiles -Q <OFFSET> -D <OUTPUT_DIR>
vol.py -f <MEMORY_DUMP> procdump -p <PID> -D <OUTPUT_DIR>
vol.py -f <MEMORY_DUMP> memdump -p <PID> -D <OUTPUT_DIR>
vol.py -f <MEMORY_DUMP> timeliner
vol.py -f <MEMORY_DUMP> iehistory
```

<details>
<summary>Details</summary>

**Description**

- Volatility 2 is the legacy version of the memory forensics framework. It requires a profile matching the OS version (determined by `imageinfo`).
- Useful for Windows XP through Windows 10 memory analysis.

**Commands**

- `imageinfo` — Suggests the correct profile for the memory dump.
- `pslist` — Lists running processes from the active process list (can be hidden by rootkits).
- `pstree` — Displays processes in a parent-child tree.
- `psscan` — Scans for processes that may be hidden or terminated.
- `psxview` — Cross-view comparison to find hidden processes.
- `netscan` — Network connections (Vista+); use `connections`/`connscan` for Windows XP.
- `cmdline` — Shows the command line arguments for each process.
- `filescan` — Scans for file objects in memory (useful for finding malware on disk).
- `dumpfiles` — Extracts a file from memory by its physical offset (from `filescan`).
- `procdump` — Dumps a process executable from memory.
- `memdump` — Dumps the full memory space of a process.
- `timeliner` — Generates a timeline of all events in the memory image.
- `iehistory` — Recovers Internet Explorer browsing history.

</details>

### 7.2 Volatility 3 <sup>· BTL1</sup>

```bash
vol.py -f <MEMORY_DUMP> windows.pslist
vol.py -f <MEMORY_DUMP> windows.pstree
vol.py -f <MEMORY_DUMP> windows.psscan
vol.py -f <MEMORY_DUMP> windows.netscan
vol.py -f <MEMORY_DUMP> windows.cmdline
vol.py -f <MEMORY_DUMP> windows.dumpfiles --pid <PID>
```

<details>
<summary>Details</summary>

**Description**

- Volatility 3 is the current version with a redesigned API and no need for manually specified profiles.
- Plugin names follow the `windows.<plugin>` naming convention.

**Commands**

- `windows.pslist` — Lists running processes from the linked list.
- `windows.pstree` — Parent-child process tree.
- `windows.psscan` — Scans for processes (includes terminated/unlinked).
- `windows.netscan` — Active network connections and listening sockets.
- `windows.cmdline` — Command line of each process.
- `windows.dumpfiles --pid <PID>` — Dumps files associated with a process.

</details>

## 8. Log Analysis & SIEM

### 8.1 Splunk <sup>· BTL1</sup>

**Search all indexes:**

```
index="*"
```

**Search for AWS EC2-related events:**

```
| search "*ec2*" amazon.aws.com
```

**Top values:**

```
| top <FIELD>
```

**Aggregate statistics by host:**

```
| stats values(CommandLine) by host
```

**Count and sort by source IP:**

```
| stats count by srcip | sort count desc
```

**Tabulate results:**

```
| table <FIELD1> <FIELD2>
```

**Deduplicate by source IP:**

```
| dedup srcip
```

<details>
<summary>Details</summary>

**Description**

- Splunk is a SIEM platform for log aggregation, searching, and alerting.
- The pipe (`|`) chains search commands: first filter, then transform, then display.
- `index="*"` searches across all indexes — use a specific index name in production to limit scope.

**Common Commands**

- `index="<INDEX>"` — Search within a specific index.
- `| stats count by <FIELD>` — Counts occurrences grouped by a field.
- `| stats values(<FIELD>) by <FIELD2>` — Lists unique values of one field grouped by another.
- `| sort count desc` — Sorts results by count in descending order.
- `| table` — Formats output as a table.
- `| dedup <FIELD>` — Removes duplicate results based on a field.

</details>

### 8.2 Sysmon & Event Logs <sup>· BTL1</sup>

**Sysmon download:**

[https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

**Windows Event Log locations:**

| Path | Description |
|---|---|
| `%WinDir%\system32\Config\*.evt` | Legacy format (XP/Server 2003) |
| `%WinDir%\system32\WinEVT\Logs\*.evtx` | Modern format (Vista+/Server 2008+) |
| `Security.evtx` | Logon events, privilege use, audit policy changes |
| `System.evtx` | Service start/stop, driver loading, system crashes |
| `Sysmon.evtx` | Process creation, network connections, file creation (Sysmon) |

<details>
<summary>Details</summary>

**Description**

- Sysmon (System Monitor) is a Windows service that logs detailed process creation, network connections, and file changes to the Windows Event Log.
- Sysmon events appear under `Applications and Services Logs/Microsoft/Windows/Sysmon/Operational`.
- Requires a configuration file for granular control over what is logged.

**Reference:** [Ultimate Windows Security Log Encyclopedia](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/default.aspx)

</details>

## 9. Network Analysis

### 9.1 Wireshark <sup>· BTL1</sup>

**Filter by IP address:**

```
ip.addr == <IP_ADDRESS>
```

**Filter by TCP port:**

```
tcp.port == <PORT>
```

**Filter by TCP window size:**

```
tcp.window_size_value >= <VALUE>
```

**Decrypt TLS traffic using Pre-Master-Secret log:**

- Set environment variable: `SSLKEYLOGFILE=<PATH>/sslkeys.log`
- In Wireshark: Edit → Preferences → Protocols → TLS → (Pre)-Master-Secret log filename → browse to the log file.

**URL-encoded form data:**

- Wireshark displays URL-encoded POST bodies as raw strings. Right-click → Show Packet Bytes to decode or copy to CyberChef for URL decoding.

### 9.2 Network Miner <sup>· BTL1</sup>

<details>
<summary>Details</summary>

**Description**

- Network Miner is a passive network forensics tool that extracts files, credentials, and metadata from PCAP captures.
- The free version only accepts `.pcap` format (not `.pcapng`). Use `Editcap` or Wireshark to convert if needed.
- Automatically reconstructs transferred files, identifies OS types, and extracts DNS queries and HTTP parameters.

</details>

## 10. Malware Analysis

### 10.1 Sandboxing Sites

| Site | Description |
|---|---|
| [Joe Sandbox](https://www.joesandbox.com/) | Deep automated malware analysis with detailed reports |
| [Hybrid Analysis](https://www.hybrid-analysis.com) | Free online sandbox by CrowdStrike with Falcon-powered detection |

### 10.2 Analysis Tools <sup>· BTL1</sup>

**BITS Parser:**

```bash
BitsParser.py --carveall > <OUTPUT_FILE>
BitsParser.py --carvedb
```

<details>
<summary>Details</summary>

**Description**

- BITS (Background Intelligent Transfer Service) is a legitimate Windows service often abused by malware for downloading payloads.
- BITS Parser extracts BITS job entries from the BITS database (`%ALLUSERSPROFILE%\Microsoft\Network\Downloader\qmgr*.dat`), revealing download URLs and file destinations.

**Parameters**

- `--carveall` — Carves all BITS job entries and redirects output.
- `--carvedb` — Carves from a specified BITS database file.

</details>

**Regshot:**

<details>
<summary>Details</summary>

**Description**

- Regshot takes a snapshot of the Windows registry, then compares two snapshots to identify changes.
- Useful for detecting registry modifications made by malware during installation or execution.
- Output is a text or HTML diff highlighting added, modified, and deleted keys and values.

</details>

**bmc-tools & RdpCacheStitcher:**

<details>
<summary>Details</summary>

**Description**

- **bmc-tools** — Parses RDP bitmap cache files (`.bmc`) often found in forensic investigations.
- **RdpCacheStitcher** — Reconstructs individual RDP bitmap cache tiles into full-screen images, revealing what an attacker saw during an RDP session.

</details>

**AUReport (Linux):**

```bash
aureport -au
aureport -if <AUDIT_LOG>
```

<details>
<summary>Details</summary>

**Description**

- AUReport generates summary reports from Linux audit logs (`auditd`).
- `-au` — Authentication report (logins, sudo usage, authentication failures).
- `-if` — Specify the input audit log file.

**Reference:** [AUReport man page](https://linux.die.net/man/8/aureport)

</details>

**ScrDec:**

<details>
<summary>Details</summary>

**Description**

- ScrDec is a Windows Script Encoder decoder — it decodes obfuscated `.vbe`/`.jse` files back to readable VBScript/JavaScript.
- If the original tool is unavailable, use CyberChef's "Windows Script Decoder" recipe as an alternative.

</details>

**Executable Types:**

| Extension | Type |
|---|---|
| `.hta` | HTML Application (executes in `mshta.exe`) |
| `.jse` | JScript Encoded File (executes in `wscript.exe` or `cscript.exe`) |
| `.vbs` | VBScript (executes in `wscript.exe` or `cscript.exe`) |

## 11. Useful Sites

| Site | Category | Description |
|---|---|---|
| [Ultimate Windows Security](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/default.aspx) | Windows Event Logs | Encyclopedia of Windows Security Log events |
| [Sysmon Download](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) | Log Analysis | Official Sysmon download page |
| [Registry Hives Reference](https://learn.microsoft.com/en-us/windows/win32/sysinfo/registry-hives) | Windows Forensics | Microsoft documentation on registry hive locations |
| [df-stream — Recycle Bin](https://df-stream.com/2016/04/fun-with-recycle-bin-i-files-windows-10/) | Windows Forensics | Blog post explaining Recycle Bin $I file format |
| [IMEI.info](https://www.imei.info/) | Android Forensics | IMEI lookup and device identification |
| [AUReport man page](https://linux.die.net/man/8/aureport) | Linux Forensics | Manual page for aureport command |
| [VirusTotal](https://www.virustotal.com) | Threat Intelligence | Multi-engine file and URL analysis |
| [Talos Intelligence](https://www.talosintelligence.com/) | Threat Intelligence | IP, domain, and file reputation |
| [URLhaus](https://urlhaus.abuse.ch) | Threat Intelligence | Malicious URL database |
| [MalwareBazaar](https://bazaar.abuse.ch/) | Threat Intelligence | Malware sample sharing |
| [PhishTank](https://www.phishtank.org/) | Phishing Analysis | Phishing URL verification |
| [Joe Sandbox](https://www.joesandbox.com/) | Malware Analysis | Automated sandbox |
| [Hybrid Analysis](https://www.hybrid-analysis.com) | Malware Analysis | CrowdStrike sandbox |
