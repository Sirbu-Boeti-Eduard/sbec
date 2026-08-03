---
description: ArchiveGate — a signature-aware, policy-driven recursive ZIP preflight inspector in Go that never extracts archive contents to disk.
---

# ArchiveGate

[ArchiveGate](https://github.com/Sirbu-Boeti-Eduard/archivegate) is a local, ZIP-only preflight inspector for untrusted archive uploads. It never extracts an entry or writes archive contents to disk, evaluating each ZIP against a JSON policy, classifying bounded entry headers, and recursively inspecting validated ZIP content even when the child archive has a misleading filename.

## Security checks

- Entry-count, declared expanded-size, and compression-ratio budgets.
- Unsafe names: absolute paths and traversal paths.
- Extension allowlist and signature/extension mismatch checks.
- Header classification for ZIP, GZIP, 7z, RAR, PE, ELF, Mach-O, PDF, PNG, JPEG, GIF, and shebang scripts.
- Configurable blocked content types; defaults block PE, ELF, Mach-O, and scripts.
- Bounded recursive ZIP inspection with depth and total-node limits.
- Encrypted entries are reported but not opened or recursed into.

The policy model follows OWASP guidance on validating ZIP paths, compression, expected type, and estimated expanded size before extraction.

## Recursion and scoring

A ZIP signature alone is not enough: ArchiveGate bounded-reads a child and parses it before recursing, so a file merely named `.zip` with non-ZIP content is not treated as a child archive. Each node has a direct score (`HIGH` 50, `MEDIUM` 20, `LOW` 5, capped at 100), and its aggregate score is the capped sum of its own direct score and all descendant direct scores. A high finding in any descendant rejects the root; any medium finding makes the root require review.

Exit codes are meaningful for scripting: `0` allow, `1` review, `2` reject, `3` input or policy error.

## Design highlights

ArchiveGate uses only Go's standard library and is documented with fixtures generated locally and harmlessly (ZIP fixtures are git-ignored and recreated with `go generate .`). Every fixture has an expected verdict and score, giving a reproducible test suite across the recursive, signature-mismatch, and blocked-content cases. The project runs `go test ./...` and `go vet ./...` in CI. MIT licensed.
