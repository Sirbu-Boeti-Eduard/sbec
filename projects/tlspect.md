---
description: tlspect — a focused Go CLI for auditing TLS certificates, protocols, ciphers, and HTTP security headers, scoring a public endpoint 0-100.
---

# tlspect

[tlspect](https://github.com/Sirbu-Boeti-Eduard/tlspect) is a local TLS configuration auditor for a single public endpoint. It turns certificate, protocol, cipher, chain, and HTTP-header evidence into a 0-100 score, a letter grade, and a prioritised remediation list.

The tool is deliberately small: it uses only the Go standard library, makes outbound connections only to the hostname supplied on the command line, and neither retains scan results nor sends them to a third party.

## Security checks

- Certificate expiry, signature algorithm, public-key type, and public-key size.
- TLS 1.0 through TLS 1.3 support, tested with a separate handshake for each version.
- A focused probe for legacy RC4 and 3DES cipher suites.
- Presence of `Strict-Transport-Security` and `Content-Security-Policy` headers.
- Whether the TLS handshake included at least one additional certificate after the leaf certificate.
- A transparent 0-100 scoring rubric and an ordered list of fixes.

## Design highlights

The terminal report uses plain status markers: ANSI-coloured `[x]`, `[!]`, and `[ ]`, with a `--no-color` flag for entirely plain ASCII output. The scanner's unit tests cover the scoring rubric and terminal-output contract, and an end-to-end test spins up a local TLS server and performs a real handshake against it, so validation is reproducible without depending on a public domain that can change its configuration.

`--fail-under` makes tlspect exit with status 1 when the score is below a supplied threshold, which lets a caller gate scripts or deployments on the result. The project includes a GitHub Actions workflow that runs `go test ./...`, `go vet ./...`, and a build on pushes and pull requests.

tlspect is an auditor, not a full TLS assessment tool: it does not enumerate every cipher suite, validate revocation, test OCSP stapling, inspect CT logs, or probe protocol vulnerabilities such as Heartbleed. MIT licensed.
