# Mutation Testing

Mutation testing evaluates the *quality* of a test suite — not just its coverage —
by introducing small, deliberate code changes (mutations) and verifying that at least
one test fails for each mutation. A mutation that is not caught by any test
("survived") reveals a gap in the test suite.

## Tools

### Rust — cargo-mutants

The Soroban smart contracts use [cargo-mutants](https://mutants.live/).

```bash
# Install (once)
cargo install cargo-mutants

# Run mutation testing on the atomic_swap contract
cd contracts/atomic_swap
cargo mutants

# Run in parallel for speed
cargo mutants -- --threads 4

# Target specific functions
cargo mutants -m "initiate_swap|accept_swap|complete_swap"
```

Full strategy and killer-test catalogue:
[`contracts/atomic_swap/MUTATION_TESTING.md`](../contracts/atomic_swap/MUTATION_TESTING.md)

### JavaScript — Stryker (planned)

A Stryker configuration for the JS SDK modules is planned. Until then, the
property-based tests (`*.property.test.js`) and fuzz tests (`*.fuzz.test.js`) provide
equivalent confidence for the batch and fee calculation modules.

## Coverage Goals

| Scope | Target mutation score |
|-------|----------------------|
| `contracts/atomic_swap` critical functions | ≥ 95% |
| `contracts/atomic_swap` overall | ≥ 85% |
| `contracts/ip_registry` critical functions | ≥ 90% |

## Running Locally

```bash
cd contracts/atomic_swap
cargo mutants -- --threads 4 --output csv > mutation_report.csv
```

Inspect survived mutations in `mutation_report.csv` and add killer tests for any
that represent real logic errors.

## Interpreting Results

| Status | Meaning | Action |
|--------|---------|--------|
| Killed | At least one test caught the mutation | ✅ None needed |
| Survived | No test caught the mutation | Write a killer test |
| Timeout | Mutation caused an infinite loop | Investigate — may indicate real bug |
| Unviable | Mutation does not compile | Ignore |

A **mutation score** below 85% on critical contract functions should block merge
(enforced manually until a CI gate is added — tracked in issue #1244).

## Writing Mutation-Killable Tests

Prefer `assert_eq!` over `assert!` — equality assertions kill far more mutations:

```rust
// ✅ Kills arithmetic and assignment mutations
assert_eq!(swap.price, 50_i128);

// ❌ Weak — survives most mutations that preserve a positive value
assert!(swap.price > 0);
```

Always test both sides of a boundary condition:

```rust
// Catches off-by-one and comparison-operator mutations
assert_eq!(result_at_zero, expected_zero);
assert_eq!(result_at_max, expected_max);
```

## Related Documentation

- [Code Coverage](code-coverage.md) — Line/branch coverage thresholds
- [Testing Strategy](testing-strategy.md) — Full test pyramid
- [Security Scanning](security-scanning.md) — CI security checks
- [`contracts/atomic_swap/MUTATION_TESTING.md`](../contracts/atomic_swap/MUTATION_TESTING.md) — Detailed atomic swap mutation strategy
- [SECURITY.md](../SECURITY.md) — Security policy
