# goodfood-ms-paiement

Payment processing microservice for the [GoodFood](https://github.com/RMurier/BAC-5-CUBE-1-COLLABORATIF) platform.

**Status:** 🚧 Scaffold — this is currently an unmodified `create-next-app` template. No Stripe integration exists yet despite `STRIPE_SECRET_KEY`/`STRIPE_WEBHOOK_SECRET` already being wired up in the environment. See [`goodfood-ms-auth`](https://github.com/RMurier/goodfood-ms-auth#readme) for what a fleshed-out service in this platform looks like.

## Table of Contents

- [Intended Purpose](#intended-purpose)
- [Tech Stack](#tech-stack)
- [Environment Variables](#environment-variables)
- [Running Locally](#running-locally)
- [Tests](#tests)
- [Security Notes](#security-notes)
- [CI/CD](#cicd)

## Intended Purpose

Handle payment intents/charges via Stripe for orders created in `ms-commandes`, plus Stripe webhook handling for payment confirmation.

## Tech Stack

- Next.js 16, React 19, TypeScript
- Tailwind CSS
- Stripe API (planned — no `stripe` package installed yet)
- Jest for tests

## Environment Variables

| Variable | Description |
|----------|--------------|
| `NODE_ENV` | `development` or `production` |
| `PORT` | Listen port (defaults to `8080` in containers) |
| `DATABASE_URL` | SQL Server connection string (database `GoodFood_Paiement_Dev` in dev) |
| `STRIPE_SECRET_KEY` | Stripe API secret key — **not yet consumed by any code** |
| `STRIPE_WEBHOOK_SECRET` | Stripe webhook signing secret — **not yet consumed by any code** |

## Running Locally

### Via the platform's docker-compose

From the [parent repo](https://github.com/RMurier/BAC-5-CUBE-1-COLLABORATIF), with `STRIPE_SECRET_KEY`/`STRIPE_WEBHOOK_SECRET` set in `.env` (test-mode Stripe keys, see [Security Notes](#security-notes)):

```bash
docker compose -f docker-compose.dev.yml up -d db-sql-dev ms-paiement-dev
```

Runs on `http://localhost:3003`.

### Standalone

```bash
npm install
npm run dev
```

## Tests

```bash
npm test
```

Currently a single sanity check ([`__tests__/sanity.test.ts`](__tests__/sanity.test.ts)), there to keep the CI test job green while the service is empty.

## Security Notes

When the Stripe integration lands, keep in mind:

- **Never** use a live (`sk_live_...`) key outside of a real production deployment. `.env.example` in the [parent repo](https://github.com/RMurier/BAC-5-CUBE-1-COLLABORATIF) intentionally only shows the `sk_test_...` / `whsec_...` placeholder shapes.
- Stripe webhook payloads must be verified against `STRIPE_WEBHOOK_SECRET` (Stripe's signature header) before being trusted — never process an unverified webhook body.
- Card data itself should never touch this service directly — use Stripe Elements/Checkout on the frontend so raw card numbers never hit our servers or logs (PCI scope reduction).

## CI/CD

Built, scanned (SonarQube, Trivy, OWASP Dependency-Check, GitGuardian) and published on every push, gated on all of them passing — see the [parent repo's CI/CD Pipeline section](https://github.com/RMurier/BAC-5-CUBE-1-COLLABORATIF#cicd-pipeline) for how the pipeline is wired across repos.
