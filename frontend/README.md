# Hallmark Web App

The frontend is planned as a mobile-first Next.js progressive web app for
checking devices and completing safer second-hand sales. See the
[repository README](../README.md) for the product scope, privacy model and
roadmap.

> **Status:** The device-registry experience described here is planned, not
> implemented. The current frontend code has not yet been aligned with this
> product scope; the flows below are not a list of available features.

## Planned user flows

- **Register a device:** validate the IMEI or serial number and compute its
  hash on the user's device before interacting with the registry.
- **Check a device:** let a buyer enter or scan an identifier and explain
  whether Hallmark has no record, a registration, a stolen-device report or a
  dispute.
- **Report a device:** let the owner submit a report through the appropriate
  contract flow, including its refundable stake.
- **Sell with escrow:** connect a Stellar wallet, deposit USDC and confirm
  handover so the escrow can release payment and transfer the registry record.

The UI must distinguish a report from a proven fact and explain that
registration and trust levels are not proof of legal ownership. A
`Registered` status is not a guarantee that a device is safe.

## Privacy and wallet expectations

- Keep the raw IMEI or serial number on the user's device; use the client-side
  hash for registry interactions.
- Explain that hashing does not make these fixed-format identifiers secret:
  someone could try candidate values against public hashes.
- Do not put names, phone numbers or addresses on-chain.
- Use Freighter for Stellar wallet interactions and Stellar Testnet during
  development.

## Development

The frontend package is in this directory. Its current package scripts include:

```bash
npm install
npm run dev
npm test
npm run build
```

These commands operate on the current frontend package; they do not imply that
the planned device registration, checking, reporting or escrow flows are
implemented.
