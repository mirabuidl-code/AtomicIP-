# TTL Management

## Overview

All persistent storage entries in the Atomic Patent smart contracts are extended with
a TTL of approximately **1 year** (6,307,200 ledgers at 5 s/ledger) on every write.

This ensures that active IP records and swap records remain accessible on-chain for at
least a year after their last modification, without requiring off-chain archival
infrastructure for normal operation.

## TTL Constants

| Constant | Value (ledgers) | Approximate duration |
|----------|-----------------|----------------------|
| `LEDGER_BUMP` | 6,307,200 | ~1 year |
| Minimum TTL threshold | 3,153,600 | ~6 months |

When a record is read, the contract checks whether the remaining TTL has fallen below
the minimum threshold and bumps it back to `LEDGER_BUMP` if so.

## Affected Storage Entries

| Entry | Storage type | TTL policy |
|-------|-------------|------------|
| `IpRecord` | Persistent | Bumped on every commit, transfer, or reveal |
| `SwapRecord` | Persistent | Bumped on initiate, accept, reveal, or cancel |
| `EscrowRecord` | Persistent | Bumped on escrow creation and release |
| `SellerSwaps` / `BuyerSwaps` index | Persistent | Bumped alongside the corresponding swap |
| Admin / Oracle config | Instance | Extended with the instance TTL on every admin write |

## Why Persistent Storage Requires Explicit TTL Management

Soroban persistent storage entries are **not** kept alive automatically — they expire
(and the data is archived) if the rent is not paid. Explicit `extend_ttl` calls are
the mechanism by which the contract pays rent, keeping entries live on the active
ledger state.

For background, see the
[Soroban Resource Limits](docs/soroban-resource-limits.md) reference and the official
[Stellar documentation on storage types](https://developers.stellar.org/docs/build/smart-contracts/storage-types).

## Operator Guidance

- **No manual intervention is required** as long as records are accessed at least once
  every ~6 months. The contract will auto-extend TTL on every read that hits the
  threshold.
- For IP records that have not been accessed in a long time, consider submitting a
  no-op `get_ip` call to trigger TTL extension before the 6-month threshold is
  crossed.
- Monitor `ip_ttl_remaining_ledgers` and `swap_ttl_remaining_ledgers` Prometheus
  metrics (exposed by the API server) to detect entries approaching expiry.

## Related Documentation

- [API Reference](docs/api-reference.md) — full contract API including TTL semantics
- [Soroban Resource Limits](docs/soroban-resource-limits.md) — per-transaction and
  per-entry storage limits
- [Performance Tuning Guide](docs/performance-tuning-guide.md) — tips on minimising
  TTL rent costs
