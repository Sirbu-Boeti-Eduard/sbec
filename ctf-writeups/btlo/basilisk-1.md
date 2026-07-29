---
description: A static malware-analysis walkthrough of a Windows PE sample, covering strings, headers, imports, packing indicators, hashing, and compile-time metadata.
---

# BTLO: Basilisk 1

<!-- Original publication date: 2024-03-17 -->

## Introduction

![Basilisk 1 investigation cover image](../../assets/images/btlo-basilisk-1-001.png)

Hello, my name is Sirbu-Boeti Eduard-Cristian and in this write-up I am going to cover the “Basilisk 1” investigation on Blue Team Labs Online.

[**Blue Team Labs Online \| Basilisk 1**<br><span>blueteamlabs.online</span>](https://blueteamlabs.online "https://blueteamlabs.online")

| Investigation detail | Description |
| --- | --- |
| Platform | Blue Team Labs Online |
| Investigation | Basilisk 1 |
| Difficulty | Easy |
| Primary evidence | A Windows Portable Executable sample named `Basilisk.bin` |
| Focus areas | String triage, PE headers, entry point, imports, packing heuristics, hashing, and timestamp analysis |
| Tools demonstrated | BinText, Exeinfo PE, PEiD, CFF Explorer, PEview, and PowerShell |

### Scenario

The lab provides a sample named `Basilisk.bin` and asks the analyst to characterize it through static analysis. The investigation focuses on what can be learned from printable strings, Portable Executable metadata, imported functions, entropy, and cryptographic identification without running the sample.

## Investigation approach

A structured static-analysis workflow moves from identification to capabilities:

1. Record the sample’s cryptographic hash so every observation can be tied to the same file.
2. Extract strings and mark unusual filenames, paths, URLs, or commands for follow-up.
3. Inspect the PE headers to establish architecture, entry point, sections, compatibility metadata, and data directories.
4. Review imported DLLs and APIs to form hypotheses about possible behavior.
5. Compare packing heuristics and treat the PE timestamp as untrusted metadata until corroborated.

Static evidence supports hypotheses about a program’s design. It does not prove that a capability was exercised during execution.

## Phase 1: Triage strings and compatibility metadata

### Question 1: Is there a string related to a suspicious executable?

The lab folder contains `Basilisk.bin` and the required analysis utilities:

![BTLO lab folder containing Basilisk.bin and the static-analysis tools](../../assets/images/btlo-basilisk-1-002.png)

Loading the sample into BinText exposes its printable strings:

![BinText results showing DLL names and other strings extracted from Basilisk.bin](../../assets/images/btlo-basilisk-1-003.png)

The output contains familiar Windows DLLs such as `kernel32.dll`, `advapi32.dll`, and `shell32.dll`, alongside the unusual string `39upd.dll`. The familiar names are expected Windows components and are not suspicious by themselves. `39upd.dll` stands out because it does not match the three modules later visible in the import table.

This makes `39upd.dll` a useful lead for further analysis, but a string alone does not prove that the file exists, is loaded, or is malicious.

> **Finding:** `39upd.dll`

### Question 2: Which Windows version does the PE header identify?

Exeinfo PE recognizes the sample as a 32-bit Windows GUI executable and exposes additional header information through its PE view:

![Exeinfo PE overview of Basilisk.bin](../../assets/images/btlo-basilisk-1-004.png)

![Exeinfo PE header view showing OS version 4.0 and the Windows NT 4.0 label](../../assets/images/btlo-basilisk-1-005.png)

The optional header contains major and minor operating-system version values of `4.0`, which the tool labels as Windows NT 4.0.

This field expresses compatibility metadata declared by the PE header. It should not be treated as proof that the sample was built or executed on that operating system.

> **Finding:** `Windows NT 4.0`

## Phase 2: Map the entry point and imported capabilities

### Question 3: What are the entry point and its section?

PEiD reports both the entry-point relative virtual address and the section containing it:

![PEiD showing entry point 00001000 and the .text section](../../assets/images/btlo-basilisk-1-006.png)

```text
Entry point RVA: 0x00001000
Entry-point section: .text
```

The entry point identifies where the loader begins executing the image after it has been mapped into memory. The `.text` section conventionally contains executable code, so this placement is consistent with a conventional PE layout.

> **Finding:** `0x00001000`, `.text`

### Question 4: What are the Import Directory RVA offset and section?

CFF Explorer parses the sample as a 32-bit Portable Executable:

![CFF Explorer overview of Basilisk.bin](../../assets/images/btlo-basilisk-1-007.png)

The Data Directories view distinguishes the field’s location in the optional header from the RVA stored inside that field:

![CFF Explorer Data Directories showing the Import Directory RVA field](../../assets/images/btlo-basilisk-1-008.png)

| Property | Value |
| --- | --- |
| Import Directory RVA field offset | `0x00000138` |
| RVA stored in the field | `0x00002050` |
| Containing section | `.rdata` |

The question asks for the offset of the Import Directory RVA field, so the requested value is `0x138`. The [Microsoft PE format specification](https://learn.microsoft.com/en-us/windows/win32/debug/pe-format) describes the import directory as the structure used to resolve references to functions exported by DLLs.

> **Finding:** `0x00000138`, `.rdata`

### Question 5: Which DLL provides `ShellExecuteA`?

The Import Directory groups imported functions beneath their source modules. Selecting `shell32.dll` reveals `ShellExecuteA`:

![CFF Explorer showing ShellExecuteA imported from shell32.dll](../../assets/images/btlo-basilisk-1-009.png)

Microsoft documents [`ShellExecuteA`](https://learn.microsoft.com/en-us/windows/win32/api/shellapi/nf-shellapi-shellexecutea) as a function that performs an operation on a specified file or object. Its presence suggests that the program may open or launch a file, application, or other shell-managed object. The import does not reveal which object or operation is used; that requires code-level or runtime analysis.

> **Finding:** `shell32.dll`

### Question 6: Which DLL provides the Registry-related imports, and what are they?

Selecting `advapi32.dll` shows four Registry APIs:

![CFF Explorer showing four Registry APIs imported from advapi32.dll](../../assets/images/btlo-basilisk-1-010.png)

- `RegCloseKey`
- `RegSetValueExA`
- `RegOpenKeyExA`
- `RegCreateKeyA`

Together, these imports indicate that the program contains code capable of creating or opening Registry keys, setting values, and closing handles. Registry modification can support legitimate configuration as well as persistence or system changes, so the surrounding call arguments and runtime behavior are needed to determine intent. Microsoft’s [Registry documentation](https://learn.microsoft.com/en-us/windows/win32/sysinfo/registry-functions) provides the semantics of these APIs.

> **Finding:** `advapi32.dll` — `RegCloseKey`, `RegSetValueExA`, `RegOpenKeyExA`, and `RegCreateKeyA`

### Question 7: Which modules are imported by Basilisk?

The module list contains three DLLs:

![CFF Explorer module list for Basilisk.bin](../../assets/images/btlo-basilisk-1-011.png)

```text
kernel32.dll
advapi32.dll
shell32.dll
```

At a high level, these imports cover core process and file functionality, Registry access, and Windows Shell operations. This is a small import set, but it is not enough on its own to classify the sample.

> **Finding:** `kernel32.dll`, `advapi32.dll`, `shell32.dll`

## Phase 3: Assess packing, establish identity, and review time metadata

### Question 8: Is Basilisk packed, and what is its entropy?

PEiD’s Extra Information window reports several heuristic results:

![PEiD packing checks showing entropy 4.63, EP Check Not Packed, and Fast Check Packed](../../assets/images/btlo-basilisk-1-012.png)

| Heuristic | Result |
| --- | --- |
| Entropy | `4.63 (Not Packed)` |
| Entry-point check | `Not Packed` |
| Fast check | `Packed` |

The entropy and entry-point checks support the lab’s expected `Not Packed` classification, while the fast check disagrees. This is a useful reminder that packer detection is heuristic. Entropy measures byte-level randomness; a lower value is less suggestive of compression or encryption, but no single threshold proves that a file is unpacked.

> **Finding:** `No`; entropy `4.63`

### Question 9: What is the SHA-256 hash of Basilisk?

PowerShell’s [`Get-FileHash`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/get-filehash) cmdlet calculates a digest from the sample’s contents:

```powershell
Get-FileHash .\Basilisk.bin -Algorithm SHA256
```

![PowerShell Get-FileHash output for Basilisk.bin](../../assets/images/btlo-basilisk-1-013.png)

```text
8DD96E84B444E5F9C0814F042DD1F679E20656354BC57F7B4A9439E66E426D66
```

The hash provides a stable identifier for this exact sample and supports evidence-integrity checks. It does not independently establish that the file is malicious.

> **Finding:** `8DD96E84B444E5F9C0814F042DD1F679E20656354BC57F7B4A9439E66E426D66`

### Question 10: Which timestamp is stored in the PE file header?

PEview exposes the parsed PE structure and raw bytes:

![PEview displaying the Basilisk.bin PE structure and raw bytes](../../assets/images/btlo-basilisk-1-014.png)

Under `IMAGE_NT_HEADERS` → `IMAGE_FILE_HEADER`, the `Time Date Stamp` field resolves to:

![PEview IMAGE_FILE_HEADER showing the timestamp 2008-10-10 15:49:18 UTC](../../assets/images/btlo-basilisk-1-015.png)

```text
2008-10-10 15:49:18 UTC
```

This is the timestamp stored in the PE header, commonly described as a compile timestamp. Because the value can be modified independently of the program’s actual build process, it should be corroborated with other metadata before being used in a timeline.

> **Finding:** `2008-10-10 15:49:18 UTC`

## Investigation outcome

| Investigative question | Finding | Analyst interpretation |
| --- | --- | --- |
| Suspicious string | `39upd.dll` | Anomalous DLL-like string requiring follow-up |
| Declared OS version | Windows NT 4.0 | PE compatibility metadata, not execution proof |
| Entry point | `0x00001000` in `.text` | Conventional executable-code location |
| Import Directory field | Offset `0x138`, section `.rdata` | Field stores RVA `0x2050` |
| Shell API module | `shell32.dll` | Provides `ShellExecuteA` |
| Registry API module | `advapi32.dll` | Provides create, open, set, and close operations |
| Imported modules | `kernel32.dll`, `advapi32.dll`, `shell32.dll` | Core Windows, Registry, and Shell capabilities |
| Packing result | Expected answer: No; entropy `4.63` | Two heuristics say not packed; fast check disagrees |
| SHA-256 | `8DD96E84B444E5F9C0814F042DD1F679E20656354BC57F7B4A9439E66E426D66` | Stable identifier for the analyzed sample |
| PE timestamp | `2008-10-10 15:49:18 UTC` | Untrusted header metadata requiring corroboration |

## Key takeaways

- Hash the sample first so every result can be tied to a specific file.
- Use strings to generate investigative leads, then confirm them through headers, imports, code, or runtime evidence.
- Distinguish a data-directory field’s file offset from the RVA stored in that field.
- Treat imported APIs as possible capabilities rather than proof of behavior.
- Compare multiple packing indicators and document disagreements instead of forcing certainty.
- Treat PE timestamps as untrusted until supported by independent evidence.

## Additional resources

- [Microsoft PE format specification](https://learn.microsoft.com/en-us/windows/win32/debug/pe-format)
- [Microsoft: `ShellExecuteA`](https://learn.microsoft.com/en-us/windows/win32/api/shellapi/nf-shellapi-shellexecutea)
- [Microsoft Registry functions](https://learn.microsoft.com/en-us/windows/win32/sysinfo/registry-functions)
- [Microsoft PowerShell: `Get-FileHash`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/get-filehash)
