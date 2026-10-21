# Security Scanning

Atomic Patent runs automated security scanning on every push and pull request in
CI/CD to detect vulnerabilities before they reach `main`.

## Tools

### Gitleaks — Secret Detection

[Gitleaks](https://github.com/gitleaks/gitleaks) scans every commit for leaked
credentials such as Stellar keypairs, API tokens, JWT secrets, database connection
strings, and HMAC keys.

Configuration: `.gitleaks.toml`

```bash
# Scan staged changes locally before committing
gitleaks detect --staged -v --exit-code 1

# Scan the full repository history
gitleaks detect --source . --verbose
```

CI stage: `gitleaks` job in `.github/workflows/ci.yml`.

### cargo-deny — Supply-Chain Policy

[cargo-deny](https://github.com/EmbarkStudios/cargo-deny) enforces:
- No crates with known security advisories (checked against the RustSec database).
- License allow-list (MIT, Apache-2.0, ISC).
- Duplicate-crate policy.

```bash
cargo deny check
```

### Clippy — Static Analysis

`cargo clippy` catches common Rust mistakes, unsafe patterns, and code quality issues.

```bash
cargo clippy --workspace -- -D warnings
```

## Running All Security Checks Locally

```bash
./scripts/security-checks.sh
```

This script runs Gitleaks, cargo-deny, Clippy, and the cargo-audit gate in sequence
and exits non-zero on any failure.

## CI Enforcement

The following checks are **required** before merging to `main`:

| Check | Tool | Job |
|-------|------|-----|
| Secret scanning | Gitleaks | `gitleaks` |
| Supply-chain policy | cargo-deny | `rust-build` |
| Static analysis | Clippy | `rust-lint` |
| Vulnerability advisory | cargo-audit | `rust-tests` |

See [SECURITY.md](../SECURITY.md) for the full security policy and incident response
procedures.

## Related Documentation

- [Dependency Scanning](dependency-scanning.md) — RustSec and Dependabot
- [Code Coverage](code-coverage.md) — Coverage thresholds enforced in CI
- [Mutation Testing](mutation-testing.md) — Test suite quality verification
- [Threat Model](threat-model.md) — Attack surface and mitigations
