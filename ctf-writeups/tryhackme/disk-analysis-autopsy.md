---
description: A forensic walkthrough of an E01 disk image using Autopsy, from evidence validation and system profiling to user-activity and threat-artifact analysis.
---

# TryHackMe — Disk Analysis & Autopsy

<!-- Original publication date: 2024-03-11 -->

## Introduction

Hello, my name is Sirbu-Boeti Eduard-Cristian and in this write-up I am going to cover the “Disk Analysis & Autopsy” room on [<span>tryhackme.com</span>](https://tryhackme.com)

[**TryHackMe \| Disk Analysis & Autopsy**<br>*TryHackMe is a free online platform for learning cyber security, using hands-on exercises and labs, all through your…*<span>tryhackme.com</span>](https://tryhackme.com/room/autopsy2ze0 "https://tryhackme.com/room/autopsy2ze0")

| Investigation detail | Description |
| --- | --- |
| Platform | TryHackMe |
| Room | Disk Analysis & Autopsy |
| Primary evidence | An E01 forensic disk image |
| Focus areas | Evidence validation, Windows artifacts, user activity, PowerShell history, and threat artifacts |
| Tools demonstrated | Autopsy case information, extracted results, keyword search, and file-content views |

> **Evidence-handling note:** In a real investigation, the source image should remain read-only. Findings should be recorded with their source path, relevant timestamps, and enough context for another analyst to reproduce them. A displayed hash is useful for identification, but it should be compared with a hash independently calculated during acquisition or verification.

## Investigation approach

The investigation moves from broad system context to progressively narrower artifacts:

1. Validate the image and record its identifiers.
2. Establish the host and user-account profile.
3. Recover network configuration from system and application artifacts.
4. Examine user-specific evidence such as bookmarks, desktop files, wallpaper, and shell history.
5. Identify security tooling, rules, and exploit-related archives.

Throughout the process, the central questions are: **Which artifact records the activity I am examining? What does the artifact establish? What additional evidence would corroborate that interpretation?**

## Phase 1: Validate the evidence and profile the system

### **What is the MD5 hash of the E01 image?**

Accessing the Case Files we can find the MD5 Hash in the HASAN2.E01

![Case information showing the forensic image hash values](../../assets/images/tryhackme-disk-analysis-autopsy-001.png)

An E01 file is an Expert Witness Format forensic image. Recording the supplied MD5 value establishes a reference identifier for the evidence; comparing it with a newly computed value helps detect an incomplete transfer or later alteration.

### **What is the computer account name?**

Accessing Results \> Operating System Information we get the computer name

![Autopsy operating-system information showing the computer name](../../assets/images/tryhackme-disk-analysis-autopsy-002.png)

Starting with Autopsy’s extracted operating-system information is efficient because it provides an immediate host identity before the investigation branches into individual accounts and artifacts.

### **List all the user accounts. (alphabetical order)**

Moving to Operating System User Account we get

![Autopsy operating-system user-account results](../../assets/images/tryhackme-disk-analysis-autopsy-003.png)

The account list provides the scope for later user-by-user review. In a production case, I would distinguish accounts recovered from the SAM registry hive from profile directories and other account references, then reconcile any differences.

### **Who was the last user to log into the computer?**

Sorting by Date Accessed we get

![Autopsy user-account table sorted by the Date Accessed field](../../assets/images/tryhackme-disk-analysis-autopsy-004.png)

Sorting provides a quick lead, but the meaning of a displayed timestamp depends on the artifact and parser. Outside the lab, I would corroborate a last-logon conclusion with Windows event logs, registry values, and profile activity rather than rely on one column alone.

## Phase 2: Reconstruct the network configuration

### **What was the IP address of the computer?**

After searching through the SYSTEM registry to no avail. I came across this article: <https://www.exploit-db.com/docs/48254>

As such, I searched for ProgramFile (x86)&#92;Look@Lan&#92;irunin.ini

![Autopsy search result and file content for the Look at LAN configuration file](../../assets/images/tryhackme-disk-analysis-autopsy-005.png)

This is a useful pivot when the expected registry path does not produce the needed value: an installed network-monitoring application may preserve its own view of the host’s interface configuration. Because application configuration can be historical, its path and timestamps should be recorded before treating the value as the system’s state at a specific time.

### **What was the MAC address of the computer? (XX-XX-XX-XX-XX-XX)**

The MAC address is on the next line of the above picture

Reading the IP and MAC address from the same artifact keeps their evidential context together. The format should be normalized only when reporting it; the original representation should remain documented.

### **What is the name of the network card on this computer?**

According to the article, we can find it in WINDOWS&#92;system32&#92;config&#92;software&#92;Microsoft&#92;Windows NT&#92;CurrentVersion&#92;NetworkCards&#92;

![Windows registry artifact containing network-card information](../../assets/images/tryhackme-disk-analysis-autopsy-006.png)

For an offline Windows image, this information is recovered from the SOFTWARE hive rather than queried from the live operating system. Capturing the exact registry key and value name avoids confusing the human-readable adapter description with a driver or service identifier.

### What is the name of the network monitoring tool?

We found the IP and MAC of the NIC in “Look@Lan”

The filename, installation path, and configuration content collectively identify the tool. This is stronger than relying on a single string appearing without context.

## Phase 3: Examine user activity and artifacts

### A user bookmarked a Google Maps location. What are the coordinates of the location?

Going to Results \> Web Bookmarks we can find it

![Autopsy web-bookmark result containing a Google Maps URL](../../assets/images/tryhackme-disk-analysis-autopsy-007.png)

The coordinates can be recovered from the bookmark URL. Preserving the full URL is important because its path, query parameters, and encoding explain exactly how the reported location was derived.

### A user has his full name printed on his desktop wallpaper. What is the user’s full name?

According to <https://www.makeuseof.com/find-desktop-wallpapers-file-location-windows-11/>

We can find the wallpaper at

`%AppData%\Microsoft\Windows\Themes\CachedFiles`

All we have to do now is to search every user and find it

![Cached Windows desktop wallpaper containing a user's name](../../assets/images/tryhackme-disk-analysis-autopsy-008.png)

Because the name was written very small in the upper left corner, it took me some tries to finally find it

For an offline image, the equivalent location is under the relevant profile, such as `Users\<username>\AppData\Roaming\Microsoft\Windows\Themes\CachedFiles`. Reviewing each profile systematically is more reliable than a global visual search, particularly when identifying text is small or partially obscured.

### A user had a file on her desktop. It had a flag but she changed the flag using PowerShell. What was the first flag?

After searching in every users’ Desktop we find

![Desktop file containing the currently stored flag](../../assets/images/tryhackme-disk-analysis-autopsy-009.png)

After searching for this file’s name we find some console history

![PowerShell console history showing the earlier flag value](../../assets/images/tryhackme-disk-analysis-autopsy-010.png)

The current file shows the result of the change; the shell history explains how it happened and preserves the earlier value. This comparison illustrates a central forensic principle: a present-state artifact can answer *what exists now*, while command history can reconstruct *what existed before and how it changed*.

### **The same user found an exploit to escalate privileges on the computer. What was the message to the device owner?**

Looking at the same user, there is a Powershell script with a Youtube link to privilege escalation, along with the flag

![PowerShell script containing a privilege-escalation reference and message](../../assets/images/tryhackme-disk-analysis-autopsy-011.png)

The script should be examined statically: record its path and hash, read embedded strings and comments, and avoid execution. The message is meaningful because it appears in the same user context as the related script and external reference.

## Phase 4: Identify security tools and threat artifacts

### 2 hack tools focused on passwords were found in the system. What are the names of these tools? (alphabetical order)

As I already know Mimikatz is a popular one, I search for it and get

![Autopsy search results for Mimikatz](../../assets/images/tryhackme-disk-analysis-autopsy-012.png)

Looking at <https://attack.mitre.org/software/> I try searching for password hacking tools and eventually get

![Autopsy search results for the second password-focused tool](../../assets/images/tryhackme-disk-analysis-autopsy-013.png)

Known tool names are effective search pivots, but filenames alone are not definitive because software can be renamed or copied into unrelated paths. In a real case, I would corroborate identification with file metadata, hashes, strings, signatures, and execution artifacts.

### There is a YARA file on the computer. Inspect the file. What is the name of the author?

Searching using regex “.\*&#92;.yar” we get

![Autopsy regular-expression search for YARA rule files](../../assets/images/tryhackme-disk-analysis-autopsy-014.png)

After searching for the file we can get the author

![YARA rule content showing its metadata and author field](../../assets/images/tryhackme-disk-analysis-autopsy-015.png)

YARA rules commonly store descriptive fields such as the author in a `meta` section. That metadata helps attribute the rule document, but it is self-declared content and should not be treated as independent proof of a person’s identity.

### One of the users wanted to exploit a domain controller with an MS-NRPC based exploit. What is the filename of the archive that you found? (include the spaces in your answer)

Searching for this exploit (<https://0xbandar.medium.com/detecting-the-cve-2020-1472-zerologon-attacks-6f6ec0730a9e>) we find out another name for it, “Zerologon”

If we try to search for it

![Autopsy search result for a Zerologon-related archive](../../assets/images/tryhackme-disk-analysis-autopsy-016.png)

We also know that %20 is “ “

Recognizing that the MS-NRPC exploit is associated with Zerologon and CVE-2020-1472 provides the search pivot. Decoding `%20` as a space reconstructs the human-readable archive name, while the encoded form should still be retained in the evidence notes.

## Findings and analytical flow

The evidence forms a connected sequence rather than a collection of isolated search results:

1. The image hash anchors the examination to a specific evidence source.
2. Host and account artifacts define the system and the users whose activity must be reviewed.
3. Registry and application configuration recover the network identity.
4. Browser, wallpaper, desktop, and PowerShell artifacts reconstruct user actions and prior content.
5. Tool names, YARA metadata, and exploit terminology provide pivots into security-related files.

This flow keeps each conclusion tied to an artifact and makes the examination reproducible. When one expected source does not answer a question, the next step is to identify another artifact created by the same operating-system feature, application, or user action.

## Investigation limitations

- This lab walkthrough relies on artifacts parsed and presented by Autopsy. Parser output should be checked against the underlying file, registry hive, or database when a conclusion is significant.
- A timestamp is not meaningful without its source field, time-zone context, and an understanding of what caused that field to change.
- Application configuration may describe a past state rather than the system state at acquisition time.
- Tool names and rule metadata are investigative leads; hashes, signatures, execution traces, and surrounding context provide stronger attribution.

## Key takeaways

- Begin with evidence integrity and system context before searching for individual answers.
- Treat user accounts as separate investigative scopes and compare artifacts across them methodically.
- Pivot between registry, application, browser, filesystem, and command-history artifacts when a single source is incomplete.
- Preserve exact paths, encoded values, and timestamps so another analyst can reproduce the reasoning.
- Examine scripts and suspected tools statically in a controlled environment; do not execute evidence from an untrusted image.

## Additional resources

- [Autopsy: Data Sources](https://www.sleuthkit.org/autopsy/docs/user-docs/3.1/ds_page.html) — supported forensic-image sources, including E01 files.
- [Microsoft: Registry Hives](https://learn.microsoft.com/en-us/windows/win32/sysinfo/registry-hives) — the Windows registry hive structure and supporting files.
- [Microsoft: about_PSReadLine](https://learn.microsoft.com/en-us/powershell/module/psreadline/about/about_psreadline?view=powershell-7.6) — PowerShell history behavior and default history-file locations.
- [MITRE ATT&CK: Software](https://attack.mitre.org/software/) — a catalog for researching security and adversary tooling.
- [YARA: Writing rules](https://yara.readthedocs.io/en/stable/writingrules.html) — rule syntax, including the `meta` section.
- [Microsoft Security Response Center: CVE-2020-1472](https://msrc.microsoft.com/update-guide/en-us/vulnerability/CVE-2020-1472) — Microsoft’s security guidance for the vulnerability associated with Zerologon.
