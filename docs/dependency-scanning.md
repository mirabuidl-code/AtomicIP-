# Dependency Scanning

Atomic Patent continuously monitors both Rust and JavaScript dependencies for known
vulnerabilities and keeps them up to date via automated tooling.

## Rust Dependencies

### cargo-audit

[cargo-audit](https://github.com/rustsec/cargo-audit) checks `Cargo.lock` against
the [RustSec Advisory Database](https://rustsec.org/) for known CVEs and
security advisories.

```bash
# Run locally
cargo audit
```

Configuration: `.cargo/audit.toml` (allowlist for accepted advisories).

CI gate: Defined in `tests/cargo_audit_gate_tests.rs` — tests fail if any un-allowed
advisory is present.

### cargo-deny

[cargo-deny](https://github.com/EmbarkStudios/cargo-deny) enforces:
- No dependencies with active security advisories.
- Approved license list.
- Duplicate crate policy.

```bash
cargo deny check
```

### Dependabot — Automated Updates

GitHub Dependabot checks for new versions of Rust (`Cargo.toml`) and JavaScript
(`package.json`) dependencies every **Monday at 03:00 UTC**.

| Update type | Behaviour |
|-------------|-----------|
| Patch (x.y.Z → x.y.Z+1) | Auto-merged after CI passes |
| Minor (x.Y → x.Y+1) | PR opened; requires review |
| Major (X.Y → X+1.Y) | PR opened; requires review and changelog check |

Configuration: `.github/dependabot.yml`.

**Guidance for reviewers:**
1. Check the changelog / release notes for breaking changes.
2. Run `cargo test --workspace` and `npm test` locally against the update.
3. Merge security patches promptly; defer non-security major bumps if release is imminent.

## JavaScript Dependencies

### npm audit

```bash
npm audit
```

Checks `package-lock.json` against the npm advisory database. Run this before
publishing or deploying.

### Dependabot (JS)

Same Dependabot configuration covers `package.json`. Auto-merge applies to
patch-level updates once CI passes.

## Running All Dependency Checks Locally

```bash
# Rust
cargo audit
cargo deny check

# JavaScript
npm audit
```

## CI Enforcement

| Check | Tool | Trigger |
|-------|------|---------|
| Rust advisory check | cargo-audit | Every push / PR |
| Rust supply-chain policy | cargo-deny | Every push / PR |
| JS advisory check | npm audit | Every push / PR |
| Automated updates | Dependabot | Weekly (Monday 03:00 UTC) |

## Related Documentation

- [Security Scanning](security-scanning.md) — Gitleaks and static analysis
- [Code Coverage](code-coverage.md) — Coverage thresholds
- [SECURITY.md](../SECURITY.md) — Full security policy and incident response
