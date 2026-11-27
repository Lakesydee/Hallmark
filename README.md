# Hallmark

> A *hallmark* is the stamp that proves a piece of metal is genuine. Hallmark does the same for second-hand phones and laptops.

**Hallmark** is an open-source ownership registry for used phones and laptops, built on [Stellar](https://stellar.org) with Soroban smart contracts. Buyers can check whether a device has been reported stolen before they pay. Honest sellers can show a clean history. Sales run through an escrow that moves the money and the ownership record together.

> **Status:** early stage. This README describes the planned scope and architecture. Items under [Roadmap](#roadmap) are not built yet, and contributions are welcome.

---

## Table of contents

- [The problem](#the-problem)
- [How Hallmark works](#how-hallmark-works)
- [Device status and trust levels](#device-status-and-trust-levels)
- [Privacy model and limits](#privacy-model-and-limits)
- [Architecture](#architecture)
- [Smart contracts](#smart-contracts)
- [Tech stack](#tech-stack)
- [Repository structure](#repository-structure)
- [Getting started](#getting-started)
- [Roadmap](#roadmap)
- [Starter issues](#starter-issues)
- [Principles](#principles)
- [Contributing](#contributing)
- [License](#license)

---

## The problem

- Stolen phones and laptops are resold in second-hand markets, and a buyer has no easy way to tell whether a device is stolen.
- Honest sellers cannot prove that their device has a clean history.
- Face-to-face sales have no protection for either side: the buyer may pay for a device that gets reclaimed, and the seller may hand over a device and not be paid.

## How Hallmark works

1. **Register.** An owner registers a device. The app computes a hash of the device's IMEI or serial number on the owner's phone, and only the hash goes on-chain, together with the owner's Stellar account.
2. **Report.** If the device is lost or stolen, the owner marks it as reported. This needs a small refundable stake, so that false reports cost something.
3. **Check.** A buyer enters or scans the IMEI or serial number before paying. Hallmark shows the status: not registered, registered, reported, or disputed.
4. **Sell with escrow.** The buyer deposits USDC into a sale escrow. After the handover, the buyer confirms and the contract releases the money to the seller and transfers the ownership record in the same transaction.
5. **Dispute.** If a sale or a report is challenged, a dispute window opens and a resolution process decides the outcome.

## Device status and trust levels

| Status | Meaning |
|---|---|
| `Unregistered` | No record exists. Hallmark cannot say anything about this device. |
| `Registered` | An owner has registered it and no one has reported it. |
| `ReportedStolen` | The owner has reported it. Listing and sale are blocked. |
| `Disputed` | A report or ownership claim is being challenged. |

Registrations carry a trust level, because software alone cannot prove that someone really owns a device:

| Trust level | How it is earned |
|---|---|
| `SelfDeclared` | The owner registered it and nothing more. |
| `RetailerAttested` | A registered retailer signed a purchase attestation. |
| `TransferVerified` | The device arrived through a completed Hallmark escrow sale. |

A `Registered` device with a higher trust level is a stronger signal, but none of these levels is proof. See the principles below.

## Privacy model and limits

- **No personal data on-chain.** Owners are pseudonymous Stellar accounts. Names, phone numbers and addresses are never stored.
- **Hashed identifiers.** Only a hash of the IMEI or serial number is stored.
- **Known limit.** IMEI numbers follow a fixed format, so someone with enough computing power could try many numbers and see which hashes exist on-chain. Hashing reduces exposure but does not make the identifiers secret. Mitigations to explore: rate-limited lookup APIs, client-side hashing with a public domain-separation value, and storing no more than the hash and status.
- **A report is a claim, not a fact.** The registry records that an owner reported a device. It does not decide who is telling the truth.

## Architecture

```
          +--------------------------+
          |  Web app (Next.js, PWA)  |
          |  QR scan, wallet login   |
          +------------+-------------+
                       |
          +------------v-------------+
          |  Backend API + indexer   |
          |  lookup API, rate limits |
          |  notifications           |
          +------------+-------------+
                       |
          +------------v-------------+
          |  PostgreSQL              |
          +------------+-------------+
                       |
          +------------v------------------------+
          |  Soroban contracts on Stellar       |
          |  device-registry | sale-escrow |    |
          |  report-stake                       |
          +-------------------------------------+
```

## Smart contracts

| Contract | Purpose |
|---|---|
| `device-registry` | Maps a device hash to its owner, status and trust level. Only the current owner can transfer or report a device. |
| `sale-escrow` | Holds the buyer's USDC and, on confirmation, pays the seller and transfers ownership in one transaction. Includes a timeout and refund path. |
| `report-stake` | Holds the stake behind a stolen report. Returns it if the report stands and slashes it if the report is overturned. |

All contracts are written in Rust for Soroban. Each one should ship with unit tests and a documented testnet deployment.

## Tech stack

- **Frontend:** Next.js (mobile-first PWA), TypeScript, Tailwind CSS, a QR scanner, Freighter wallet
- **Backend:** Node.js with TypeScript, PostgreSQL, SEP-10 authentication, event indexer
- **Blockchain:** Stellar, Soroban smart contracts (Rust), USDC payments

## Repository structure

Planned monorepo layout:

```
hallmark/
├── apps/
│   ├── web/            # Next.js PWA
│   └── api/            # backend API and event indexer
├── contracts/
│   ├── device-registry/
│   ├── sale-escrow/
│   └── report-stake/
├── packages/
│   └── sdk/            # TypeScript client for the contracts
├── docs/
└── README.md
```

## Getting started

The project scaffolding is still being set up. Once it exists, the setup is expected to look like this:

```bash
git clone https://github.com/<your-org>/hallmark.git
cd hallmark

# contracts
cd contracts/device-registry
cargo test

# web app
cd ../../apps/web
npm install
npm run dev
```

Prerequisites: Node.js 20+, Rust with the `wasm32v1-none` target, the [Stellar CLI](https://developers.stellar.org/docs/tools/cli), and the [Freighter](https://www.freighter.app/) wallet. Use **Stellar Testnet** during development.

## Roadmap

**Phase 1: registry**
- [ ] Monorepo scaffolding and CI
- [ ] `device-registry` contract with tests
- [ ] Client-side hashing of IMEI and serial numbers
- [ ] Wallet login (SEP-10)

**Phase 2: check and report**
- [ ] Buyer check page with QR scanning
- [ ] `report-stake` contract and stolen-report flow
- [ ] Rate-limited lookup API

**Phase 3: safe sales**
- [ ] `sale-escrow` contract with timeout and refund
- [ ] Seller listing and buyer confirmation flow
- [ ] Ownership transfer inside the escrow transaction

**Phase 4: trust and disputes**
- [ ] Retailer attestation for `RetailerAttested` registrations
- [ ] Dispute window and a simple resolution panel
- [ ] Optional reference to an official police report

## Starter issues

Good places to begin (labels in brackets):

- Write the `device-registry` contract with owner-only transfer and report functions `[contracts]` `[good first issue]`
- Add tests that block listing a `ReportedStolen` device `[contracts]` `[good first issue]`
- Implement the `sale-escrow` deposit, confirm and refund paths `[contracts]`
- Add a timeout that lets a buyer reclaim funds if the seller never confirms `[contracts]`
- Implement stake locking and slashing in `report-stake` `[contracts]`
- Build the client-side IMEI and serial hashing helper with test vectors `[sdk]` `[good first issue]`
- Add IMEI format validation (length and check digit) `[sdk]` `[good first issue]`
- Build the SEP-10 login endpoint `[backend]`
- Create the Soroban event indexer for registry and escrow events `[backend]`
- Add rate limiting to the lookup API `[backend]`
- Build the buyer check page with a QR scanner `[frontend]`
- Write the contributor guide and local setup docs `[docs]` `[good first issue]`

## Principles

- **A report is a claim.** Hallmark records what owners say. It never labels a person a thief and never replaces a police report.
- **False reports must cost something.** The stake exists to discourage abuse, and there must always be a way to dispute a report.
- **Registration is not proof of ownership.** Trust levels show how much evidence backs a registration, and the interface must say so clearly.
- **Collect as little as possible.** No personal data on-chain, and no more off-chain than needed.
- **Support the law, not replace it.** Any link to official reports or police agencies is optional and follows local rules.

## Contributing

1. Fork the repository and create a branch for your change.
2. Pick an issue, or open one to discuss your idea first.
3. Write tests for contract and backend changes.
4. Open a pull request that explains what you changed and why.

Please be respectful and keep discussions focused on the work. A `CODE_OF_CONDUCT.md` and `CONTRIBUTING.md` will be added with the first release.

## License

Released under the [MIT License](LICENSE).
