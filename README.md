# Ticketing API — Test Automation

[![CI](https://github.com/sush-dev-git/ticketing-api-automation/actions/workflows/ci.yml/badge.svg)](https://github.com/sush-dev-git/ticketing-api-automation/actions/workflows/ci.yml)

How I structure **API test automation**. The suite runs against a small,
self-hosted mock **ticket-purchase service** (Node + Express) so the tests can
exercise things a public placeholder API can't: real state, validation,
auth-gated writes, and idempotent purchases.

> **Why a self-hosted service instead of a public test API?** Testing a service
> I control means the suite is fully deterministic — no third-party rate limits,
> no secrets, no flaky network in CI — and it unlocks the *interesting* tests
> (state lifecycle, idempotency, header propagation) that a read-only mock API
> simply can't express. The service is deliberately thin; the test architecture
> is the point.

## What this demonstrates

- **Schema-as-contract validation.** Every response is checked against a JSON
  Schema via a custom `toMatchSchema` (ajv) matcher — a renamed field or changed
  type fails loudly in the suite instead of silently breaking a consumer.
- **In-process, deterministic testing.** SuperTest drives the Express app
  directly (no port binding, no network), so the suite is fast and reproducible.
  Set `BASE_URL` to point the *same* suite at a deployed instance for smoke tests.
- **Dependency-injected app/store.** The app is a factory with an injectable
  store, so every test runs against clean, isolated state.
- **Negative and edge cases as first-class citizens** — not just the happy path.

## The system under test

A mock ticketing service with entirely invented data (fictional events, venues,
and cities). Endpoints land incrementally:

| Endpoint | Status |
| --- | --- |
| `GET /events` (paginated, filterable) | ✅ |
| `GET /events/:id` | ✅ |
| `POST /auth/login` (bearer token) | planned |
| `POST /events/:id/holds` (seat holds with TTL) | planned |
| `POST /orders` (idempotent purchase, oversell prevention) | planned |

## Quick start

```bash
npm install
npm test            # run the suite in-process (no network)
npm start           # run the mock service on http://localhost:3000
```

## Project structure

```
ticketing-api-automation/
├── server/                   # the mock service (system under test)
│   ├── app.js                # Express app factory (injectable store)
│   ├── server.js             # entry point for `npm start`
│   ├── store.js              # in-memory store + seed data
│   ├── config.js             # service config (PORT)
│   └── routes/
│       └── events.js         # events read endpoints
├── tests/
│   ├── helpers/
│   │   ├── client.js         # SuperTest client (in-process or BASE_URL)
│   │   └── schemas.js        # JSON Schemas + ajv validation helper
│   └── suites/
│       └── events.test.js    # events read-path coverage
├── jest.setup.js             # registers the toMatchSchema matcher
└── .github/workflows/ci.yml
```

## License

MIT — see [LICENSE](LICENSE).
