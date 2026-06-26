# Ticketing API — Test Automation

[![CI](https://github.com/susheela-hinchig/ticketing-api-automation/actions/workflows/ci.yml/badge.svg)](https://github.com/susheela-hinchig/ticketing-api-automation/actions/workflows/ci.yml)

API test automation for a small mock ticket-purchase service (Node + Express).
The service is self-hosted so the tests can cover things a public placeholder API
can't: real state, validation, auth-gated writes, and idempotent purchases.

I went with a service I control rather than a public test API for two reasons.
First, the suite stays deterministic in CI: no third-party rate limits, no
secrets, no flaky network. Second, it makes room for the tests that actually
matter here (state lifecycle, idempotency, header propagation), which a
read-only API can't express. The service itself is kept small on purpose.

## API docs

The API is described by an OpenAPI spec ([`openapi.yaml`](openapi.yaml)), which
is the single source of truth for the contract.

- **Browse online:** https://susheela-hinchig.github.io/ticketing-api-automation/
  (rendered with Redoc, read-only — the mock service isn't deployed, so there's
  no live backend to call).
- **Run locally:** `npm start`, then open http://localhost:3000/docs for Swagger
  UI, or http://localhost:3000/openapi.json for the raw spec.

## What's covered

- **Schema validation.** Every response is checked against a JSON Schema with a
  custom `toMatchSchema` (ajv) matcher, so a renamed field or changed type is
  caught in the suite rather than by a downstream consumer.
- **In-process tests.** SuperTest drives the Express app directly (no port, no
  network), which keeps runs fast and repeatable. Set `BASE_URL` to run the same
  suite against a deployed instance for smoke tests.
- **Injected store.** The app is a factory with an injectable store, so each test
  runs against clean, isolated state.
- **Negative and edge cases**, not just the happy path.

## The service under test

A mock ticketing service with invented data (made-up events, venues, and cities).
Endpoints are being added in stages:

| Endpoint | Status |
| --- | --- |
| `GET /events` (paginated, filterable) | done |
| `GET /events/:id` | done |
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
├── server/                   # the mock service under test
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
│       └── events.test.js    # events read tests
├── openapi.yaml              # OpenAPI spec (single source of truth)
├── docs/
│   └── index.html            # Redoc page published to GitHub Pages
├── jest.setup.js             # registers the toMatchSchema matcher
└── .github/workflows/
    ├── ci.yml                # runs the test suite
    └── pages.yml             # publishes the docs to GitHub Pages
```

## License

MIT — see [LICENSE](LICENSE).
