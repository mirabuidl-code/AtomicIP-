# Code Coverage

Atomic Patent enforces minimum code coverage thresholds on every CI run to ensure
the test suite exercises a meaningful fraction of the codebase.

## JavaScript Coverage

The JavaScript SDK and batch operations layer uses [Jest](https://jestjs.io/) with
built-in coverage collection.

### Running Coverage Locally

```bash
# Generate coverage report (written to coverage/)
npm run test:coverage

# View the HTML report
open coverage/lcov-report/index.html
```

### Thresholds

Thresholds are configured in `scripts/coverage-config.json` and enforced by
`scripts/coverage-threshold-gate.sh`.

| Metric | Minimum threshold |
|--------|-------------------|
| Statements | 80% |
| Branches | 75% |
| Functions | 80% |
| Lines | 80% |

The threshold gate script runs as part of CI and fails the build if any metric
falls below its minimum.

```bash
# Run the gate locally
./scripts/coverage-threshold-gate.sh
```

Coverage results are also uploaded to [Codecov](https://codecov.io/gh/AtomicIP/AtomicIP-)
for trend tracking across PRs and branches.

### Coverage Report in CI

The `JavaScript Tests` job collects coverage with `--coverage` and stores the
`lcov.info` report as an artifact. The gate runs immediately after.

## Rust Coverage

Rust test coverage is tracked via `cargo-tarpaulin` or `llvm-cov`. To generate a
local report:

```bash
# Using cargo-tarpaulin (install once)
cargo install cargo-tarpaulin

cargo tarpaulin --workspace --out Html --output-dir coverage/rust
open coverage/rust/tarpaulin-report.html
```

Rust coverage is not currently gated in CI but is tracked informally. A formal
threshold will be added in a future issue.

## Writing Coverage-Positive Tests

- **Test every branch.** If a function has an early `return` on error, write a test
  that exercises that branch.
- **Test exported public API.** Uncovered exports indicate dead code or missing
  integration tests.
- **Do not write coverage-only tests.** Tests should assert correctness; coverage is a
  side effect, not the goal.

## Related Documentation

- [Testing Strategy](testing-strategy.md) — Overall test pyramid and strategy
- [Mutation Testing](mutation-testing.md) — Verify tests actually catch regressions
- [Security Scanning](security-scanning.md) — CI security checks
- [SECURITY.md](../SECURITY.md) — Full security policy
