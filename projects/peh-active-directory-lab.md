---
description: peh-ad-lab — automated, isolated Active Directory lab for TCM Security's Practical Ethical Hacking course, built from raw ISOs to a configured MARVEL.local domain via VMware Tools orchestration.
---

# Practical Ethical Hacking — Automated AD Lab (peh-ad-lab)

[peh-ad-lab](https://github.com/Sirbu-Boeti-Eduard/peh-ad-lab) is a PowerShell orchestrator that builds the Active Directory lab used in TCM Security's Practical Ethical Hacking (PEH) course — the environment behind the PJPT and PNPT certification pathways — from raw evaluation ISOs to a fully configured `MARVEL.local` domain, without touching a single Windows installer. It is not affiliated with or endorsed by TCM Security.

## Lab topology

Three VMs on an isolated `VMnet1` host-only network with no NAT, bridged adapter, or gateway:

| VM | OS | IP | Role |
| --- | --- | --- | --- |
| `HYDRA-DC` | Windows Server 2022 | 10.0.1.10 | AD DS, DNS, AD CS, file server |
| `ThePunisher` | Windows 10 | 10.0.1.20 | Frank Castle's domain client |
| `Spiderman` | Windows 10 | 10.0.1.21 | Peter Parker's domain client |

An optional Kali attack box attaches with a second NAT NIC, so the lab network stays reachable only from inside it. The environment deliberately reproduces the course's weak credentials, privileged service account, and open `hackme` share — it is an intentionally vulnerable training lab.

## Orchestration

PowerShell drives VMware through `vmrun` and VMware Tools guest operations — guest PowerShell, file transfer, and Tools-readiness probes — so the build knows when a guest is genuinely ready rather than merely powered on. The build follows a bow-tie dependency: three concurrent Windows installs, then the domain controller creates and verifies the domain, then two concurrent client configurations and joins.

The domain-controller workflow installs AD DS, DNS, and AD CS, promotes a new `MARVEL.local` forest, and creates the course's users, groups, service-account SPN, and shared folder; the client workflow joins both machines to the domain, enables the local administrators the exercises need, and maps the drive the course expects. A `-BaseOnly` switch stops after the domain controller, and `-Unattended` auto-answers the manual restart checkpoints.

## Design highlights

The build is resumable and self-healing: `state.json` records each machine's stage, reboots after promotion and joins are restarted automatically if a guest stays unresponsive, leftover VMware lock files are cleared on every start, and domain configuration is gated until the AD services are actually answering. A Pester test suite covers profile validation, state transitions, and build-resume guards, and runs in CI on every push. The project originally ran on VirtualBox but moved to VMware Workstation after VirtualBox could not reliably install the VMs. MIT licensed.
