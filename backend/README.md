# Hallmark Backend

The backend is planned to support Hallmark's ownership registry for second-hand
phones and laptops. The product scope and current project status are described
in the [repository README](../README.md).

> **Status:** The device-registry backend described here is planned, not
> implemented. The current backend code has not yet been aligned with the
> device-registry scope; do not treat the responsibilities below as available
> API functionality.

## Planned responsibilities

- Provide a lookup API for device hashes and their on-chain status.
- Apply rate limits to lookups. IMEI and serial number formats have a small
  search space, so hashing alone does not prevent enumeration.
- Authenticate Stellar accounts using SEP-10 where an authenticated action is
  required.
- Index Soroban contract events for the application and its notifications.
- Use PostgreSQL for off-chain application and indexing data. Keep personal
  data out of the chain and collect only the off-chain data needed to operate
  the service.

The chain remains the source of truth for device ownership, status and trust
level. The backend should not turn an owner's stolen-device report into an
independently verified fact.

## Planned integration

The web app will send a client-computed device hash to the lookup API. The API
will return the available registry status; it should not require an IMEI or
serial number in plaintext. Registration, reporting and ownership changes
will be authorized by the user's Stellar account and submitted to the
appropriate Soroban contract.

The backend may index contract events and provide notifications, but it must
not claim that a device is safe or that a report is proven. A missing registry
record means only that Hallmark has no record for the submitted hash.

## Development

The backend package is in this directory. Its current package scripts include:

```bash
npm install
npm run start:dev
npm test
npm run build
```

These commands operate on the current backend package; they do not imply that
the planned device lookup, SEP-10 or Soroban indexer functionality is
implemented.
