---
description: email-dumper — automated email dumping for penetration testing and CTFs, saving every IMAP or POP3 message as .eml files.
---

# email-dumper

[email-dumper](https://github.com/Sirbu-Boeti-Eduard/email-dumper) automates mailbox dumping for penetration testing and CTFs. Given valid credentials, it dumps every message from an IMAP or POP3 mailbox — including SSL/TLS variants — to `.eml` files for offline review.

## Features

- **Dual protocol support** — works with both IMAP and POP3 servers.
- **Every encryption mode** — plaintext, implicit TLS/SSL, and STARTTLS on both protocols.
- **Full message fidelity** — fetches raw RFC822 messages saved as standard `.eml` files, organised per mailbox folder.
- **Zero external dependencies** — standard library only (`imaplib`, `poplib`, `ssl`), no package management needed.
- **Single-file, drop-in ready** — one Python script, run it anywhere.
- **Quiet and verbose modes** — go stealthy with `-q` or debug with `-v`.

The tool grew out of the pain of manually dumping mailboxes over `openssl s_client` or `nc` with raw IMAP/POP3 commands — slow, repetitive, and error-prone when a server exposes dozens of folders. A single command turns valid credentials into a complete offline copy of a mailbox.

## Design highlights

The script handles protocol quirks found in real labs and CTFs: IMAP `\Noselect` container folders are skipped automatically, STARTTLS-on-the-plaintext-port servers are covered with a `--starttls` flag, and folder names are sanitised for filesystem safety.

The repository includes a reproducible Dovecot test server via `docker compose` that exercises plaintext, implicit TLS, STARTTLS, POP3, self-signed certificates, and nested mailboxes. A seed script generates varied messages deterministically — plaintext, HTML, multipart with attachments, and a hidden CTF flag for verification — so the full test suite (15 tests across all protocol/encryption combos) produces identical results on every run and in CI. MIT licensed.
