# Hallmark Soroban Contracts

Hallmark's planned contracts will provide the on-chain ownership registry and
the atomic settlement primitives for second-hand device sales. See the
[repository README](../README.md) for the product scope and roadmap.

> **Status:** The device-registry contracts described here are planned, not
> implemented. The current contracts workspace contains freight-oriented
> contracts and does not yet provide the device registry, sale escrow or
> report stake described below.

## Planned contracts

| Contract | Planned responsibility |
|---|---|
| `device-registry` | Map a client-computed device hash to its owner, status and trust level. Only the current owner can report or transfer the device. |
| `sale-escrow` | Hold the buyer's USDC and, after handover confirmation, pay the seller and transfer the ownership record in the same transaction. Include a timeout and refund path. |
| `report-stake` | Hold the refundable stake associated with a stolen-device report; return or slash it according to the dispute outcome. |

The registry should store a hash rather than a raw IMEI or serial number, and
must treat a report as a claim rather than a finding of fact. Trust levels
describe the evidence behind a registration; they do not prove ownership.

## Development expectations

The planned contracts will be written in Rust for Soroban. Each contract
should include unit tests and documented Stellar Testnet deployment steps.
Use the Rust toolchain and target specified by the contracts workspace, plus
the [Stellar CLI](https://developers.stellar.org/docs/tools/cli).

When the planned contract crates are available, run their tests from the
workspace:

```bash
cd contracts
cargo test
```

The existing workspace and its contract READMEs document the current
freight-oriented contracts. They are not documentation for the planned
device-registry contracts.
