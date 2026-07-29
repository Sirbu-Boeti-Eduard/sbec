---
description: A methodical phishing-email investigation covering header analysis, originating infrastructure, sender-domain controls, and safe attachment triage.
---

# TryHackMe — The Greenholt Phish

<!-- Original publication date: 2024-03-13 -->

## Introduction

Hello, my name is Sirbu-Boeti Eduard-Cristian and in this write-up I am going to cover “The Greenholt Phish” room on [<span>tryhackme.com</span>](https://tryhackme.com)

[**TryHackMe \| The Greenholt Phish**<br>*TryHackMe is a free online platform for learning cyber security, using hands-on exercises and labs, all through your…*<span>tryhackme.com</span>](https://tryhackme.com/room/phishingemails5fgjlzxc "https://tryhackme.com/room/phishingemails5fgjlzxc")

| Investigation detail | Description |
| --- | --- |
| Platform | TryHackMe |
| Room | The Greenholt Phish |
| Primary evidence | An email message and its attachment |
| Focus areas | Email triage, header analysis, infrastructure enrichment, SPF and DMARC, static attachment triage |
| Tools demonstrated | Email source viewer, CyberChef, WHOIS, DNS lookup services, and Linux command-line utilities |

## Investigation approach

A useful phishing-analysis workflow moves from low-risk observations to progressively deeper evidence:

1. Preserve the original message and attachment.
2. Triage the subject, visible sender, reply address, and attachment metadata.
3. Inspect the raw message source rather than relying only on the email client.
4. Trace the message infrastructure using header fields and independent registration data.
5. Examine the sender domain's SPF and DMARC records.
6. Hash and identify the attachment without executing it.
7. Separate confirmed observations from assumptions and attribution claims.

Throughout the process, the central questions are: **What am I trying to prove? Which artifact contains that evidence? Can it be manipulated? What independent evidence can corroborate it?**

## Phase 1: Initial email triage

Initial triage establishes what the message claims to be. These visible fields are useful starting points, but they should not be treated as proof of identity because an attacker can manipulate many of them.

### 1. What is the **Transfer Reference Number** listed in the email’s **Subject**?

Simply open the email and check it:

![Email content showing the interbank transfer reference field in context.](../../assets/images/tryhackme-the-greenholt-phish-001.jpeg)

The subject and message body provide the first pieces of context. In a real investigation, record the value exactly as displayed, but also preserve the original message so it can later be compared with the raw `Subject` header.

### 2. Who is the email from?

### 3. What is his email address?

### 4. What email address will receive a reply to this email?

These three questions examine different parts of the sender identity. The display name answers who the message claims to be from, the address in the `From` field identifies the claimed author mailbox, and `Reply-To` can redirect responses somewhere else.

We can get all the info from the headers of the email:

![Email header view showing the From, Subject, and Reply-To fields.](../../assets/images/tryhackme-the-greenholt-phish-002.jpeg)

| Header field | Meaning | Investigative value |
| --- | --- | --- |
| `From` display name | Human-readable identity shown by the mail client | Easy to imitate; useful for identifying impersonation |
| `From` address | Mailbox presented as the message author | Compare its domain with the organization being impersonated |
| `Reply-To` | Address a mail client should use for replies | A mismatch can redirect a victim away from the apparent sender |
| `Return-Path` | Envelope return address recorded during delivery | Used later for delivery failures, SPF context, and domain analysis |

A mismatch between these fields is not automatically malicious, because legitimate mailing services also use different addresses. It is nevertheless an important anomaly that should be explained and corroborated with the delivery path and authentication results.

## Phase 2: Tracing the originating infrastructure

The visible sender identity is only one layer of the message. Raw headers contain routing and metadata fields that can help reconstruct how the email reached the recipient.

### 5. What is the Originating IP?

Using Cyberchef to extract IPs and checking the source email file we get:

![CyberChef output and raw email source showing the originating-IP field.](../../assets/images/tryhackme-the-greenholt-phish-003.png)

[CyberChef](https://gchq.github.io/CyberChef/) is useful for extracting candidate IP addresses from a large header block, but extraction is only the first step. Each address must be interpreted in the context of the field that contains it.

- `Received` fields describe message-transfer hops and are normally added as the message moves between mail systems.
- `X-Originating-IP` is a non-standard field that some providers add to identify the client that submitted a message.
- Addresses belonging to private networks, security gateways, or the recipient's own mail infrastructure may not identify the original sender.

Read a `Received` chain carefully and identify which systems are trusted before relying on a value. A header supplied outside the trusted mail path can be forged, so corroboration matters.

> **Corroboration step:** Treat the originating-IP field as one data point. Compare it with the earliest credible `Received` hop, timestamps, and any available `Authentication-Results` header.

### 6. Who is the owner of the Originating IP? (Do not include the “.” in your answer.)

Using <https://whois.domaintools.com> on the previous IP we get:

![WHOIS result showing registration details for the originating IP address.](../../assets/images/tryhackme-the-greenholt-phish-004.jpeg)

A WHOIS or Regional Internet Registry lookup enriches the IP address with its registered organization, network range, and allocation details. This helps distinguish a cloud provider, hosting company, residential ISP, corporate network, or security service.

> **Attribution caveat:** The registered network owner is not necessarily the sender or attacker. It identifies who controls or receives the address allocation, not who operated a particular system at the time of the email.

For independent verification, an analyst can also query the relevant Regional Internet Registry or use a command-line lookup:

```bash
whois <originating-ip>
```

## Phase 3: Evaluating sender-domain controls

After identifying the `Return-Path` domain, the next step is to examine its published email-authentication policy. SPF and DMARC provide useful context, but a DNS record alone does not prove whether this specific message passed authentication.

### 7. What is the SPF record for the Return-Path domain?

First let’s find the Return-Path in the source of the email:

![Raw email header showing the Return-Path field.](../../assets/images/tryhackme-the-greenholt-phish-005.jpeg)

Next, using <https://easydmarc.com/tools/spf-lookup> we can get the SPF record:

![SPF lookup result for the Return-Path domain.](../../assets/images/tryhackme-the-greenholt-phish-006.jpeg)

Sender Policy Framework (SPF) is published as a DNS TXT record. It lists the systems or included policies authorized to send mail using a domain's envelope sender identity.

The same lookup can be reproduced directly from a terminal:

```bash
dig +short TXT <return-path-domain>
```

Common SPF mechanisms include `ip4`, `ip6`, `a`, `mx`, and `include`. The final qualifier—such as `-all` or `~all`—describes how receivers should treat senders that do not match an authorized mechanism.

> **Interpretation:** Finding the SPF policy documents the domain's declared sending infrastructure. To determine what happened to this particular email, inspect the trusted `Authentication-Results` header or evaluate the connecting IP against the policy.

### 8. What is the DMARC record for the Return-Path domain?

Using the same website as above, but with the DMARC checking service we get:

![DMARC lookup result for the Return-Path domain.](../../assets/images/tryhackme-the-greenholt-phish-007.jpeg)

Domain-based Message Authentication, Reporting, and Conformance (DMARC) builds on SPF and DKIM. It tells receivers how the visible `From` domain expects authentication alignment failures to be handled and can specify reporting destinations.

DMARC is queried at the `_dmarc` subdomain:

```bash
dig +short TXT _dmarc.<domain>
```

Useful tags include:

- `v=DMARC1` — identifies the record as DMARC.
- `p` — requests a policy such as `none`, `quarantine`, or `reject`.
- `pct` — defines the percentage of messages to which the policy applies.
- `rua` — provides an address for aggregate reports.

The existence of a DMARC record is not evidence that a message passed DMARC. The message's trusted authentication results and domain alignment must still be examined.

## Phase 4: Examining the attachment safely

The attachment should be treated as a separate evidence item. Preserve its original name, work on a copy, calculate a cryptographic hash, record its exact size, and identify its content from file signatures before considering any deeper analysis.

### 9. What is the name of the attachment?

We can see the attachment here at the bottom of the mail:

![Email client showing the message attachment and its filename.](../../assets/images/tryhackme-the-greenholt-phish-008.jpeg)

The filename is useful for documentation and correlation, but it is attacker-controlled metadata. A convincing name or extension does not establish the attachment's real content.

### 10. What is the SHA256 hash of the file attachment?

Going to terminal and using sha256sum on the file attachment we get:

![Terminal output showing the SHA-256 calculation for the attachment.](../../assets/images/tryhackme-the-greenholt-phish-009.jpeg)

```bash
sha256sum <attachment>
```

SHA-256 provides a stable identifier for the exact file contents. Record it before making changes so results from other tools, sandboxes, or threat-intelligence sources can be correlated with the same sample.

### 11. What is the attachments file size? (Don’t forget to add “KB” to your answer, **NUM KB**)

Paying attention to the following: “**The binary system measures a kilobyte as 1,024 bytes, whereas the decimal system measures a kilobyte as an even 1,000 bytes**.”

We shall take the binary system measurement. But before this we get the number of bytes

![Terminal output showing the attachment's size in bytes.](../../assets/images/tryhackme-the-greenholt-phish-010.jpeg)

![Byte-to-kilobyte conversion showing decimal and binary results.](../../assets/images/tryhackme-the-greenholt-phish-011.jpeg)

The byte count can be collected without opening the attachment:

```bash
stat -c '%s bytes' <attachment>
```

Using the binary conversion:

```text
reported value = bytes / 1024
```

Strictly speaking, a 1,024-byte unit is a kibibyte (KiB), while a decimal kilobyte (KB) is 1,000 bytes. When documenting results, record the original byte count and state which conversion was used.

### 12. What is the actual file extension of the attachment?

Using the “file” command we can get the true file type

![Terminal output from the file command identifying the attachment's content type.](../../assets/images/tryhackme-the-greenholt-phish-012.jpeg)

```bash
file <attachment>
xxd -l 16 <attachment>
```

The `file` utility compares the file's contents with known signatures instead of trusting its name. Examining the first bytes provides an additional way to confirm whether the magic value is consistent with the detected type.

## Findings and analytical flow

| Investigation phase | Evidence examined | Outcome |
| --- | --- | --- |
| Initial triage | Subject, display name, `From`, and `Reply-To` | Recorded the message's claims and compared its sender identities |
| Infrastructure tracing | Raw headers, originating IP, and WHOIS | Identified candidate source infrastructure and enriched its registration context |
| Domain controls | `Return-Path`, SPF, and DMARC | Documented the sender domain's published mail policy and its limitations |
| Attachment triage | Filename, SHA-256, byte size, and file signature | Established reproducible identifiers and the attachment's apparent content type without executing it |

The flow matters because each stage builds on evidence collected earlier. Visible sender information produces hypotheses. Raw headers provide routing context. Infrastructure and DNS lookups add independent context. Static attachment triage then records the file safely and determines which deeper analysis techniques would be appropriate.

## Key takeaways

1. Start with visible message content, but verify important details in the raw source.
2. Compare sender-related fields rather than evaluating each field in isolation.
3. Treat extracted IP addresses as candidates that require header context and corroboration.
4. Separate infrastructure ownership from attacker attribution.
5. Interpret SPF and DMARC alongside trusted authentication results.
6. Hash and identify attachments before opening or executing them.
7. Document both the result and the reasoning that produced it.

## Additional resources

- [RFC 5322: Internet Message Format](https://www.rfc-editor.org/rfc/rfc5322.html) — structure and syntax of internet messages and header fields.
- [RFC 7208: Sender Policy Framework](https://www.rfc-editor.org/rfc/rfc7208.html) — SPF processing and record syntax.
- [RFC 7489: DMARC](https://www.rfc-editor.org/rfc/rfc7489.html) — DMARC policy, alignment, and reporting concepts.
- [RFC 8601: Authentication-Results](https://www.rfc-editor.org/rfc/rfc8601.html) — standardized reporting of message-authentication results.
- [CyberChef](https://gchq.github.io/CyberChef/) — browser-based transformations and extraction workflows.
