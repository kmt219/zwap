<div align="center">

# ZWAP

### Solana to Zcash bridge prototype

A Next.js interface, Solana program and centralized relayer for exploring SOL/USDC deposits and shielded Zcash payouts.

[Local setup](docs/SETUP.md) · [Architecture](docs/ARCHITECTURE.md) · [Deployment cautions](DEPLOYMENT.md) · [Source & credits](docs/PROVENANCE.md)

</div>

> **Demo software. Do not send real funds.** This import has not been built, deployed, audited or tested end to end. The source includes simulated transactions and fixed exchange rates. A working interface is not proof of settlement or privacy.

## What's in the code

- Next.js 16 / React 19 interface with wallet connection, deposit form and status pages.
- tRPC routers for deposits, quotes, address generation, health information and receipts.
- PostgreSQL persistence through Drizzle ORM.
- An Anchor Solana program and TypeScript client for SOL and USDC deposit flows.
- A relayer that listens for Solana program events and calls a Zcash RPC client.
- Optional in-process demo status simulation, fee estimates and signed receipt generation.

These describe source components, not verified operational guarantees. In particular, this is not an atomic swap or a trustless bridge.

## Start with local development

Use Bun 1.2.23 (the version declared in `package.json`), Docker Compose and a Node runtime compatible with Next.js 16. Start with a local database and demo mode, not a funded wallet or mainnet endpoint.

```bash
git clone https://github.com/kmt219/zwap.git
cd zwap
bun install
cp .env.example .env
cp apps/web/.env.example apps/web/.env.local
```

Before starting, set `DATABASE_URL` in both environment files to match `docker-compose.yml`, and set `DEMO_MODE=true` and `NEXT_PUBLIC_DEMO_MODE=true` in `apps/web/.env.local`. See the [full setup guide](docs/SETUP.md) for the exact local-only values and limitations.

```bash
docker compose up -d
bun run db:push
bun run dev:web --env-mode=loose
```

The declared development script serves the frontend at **http://localhost:3001**, not port 3000. Do not start the relayer for the demo-only path. These commands are source-reviewed instructions, not a verified successful run.

## How the pieces fit

```text
Wallet + Next.js interface
           |
       tRPC routers ------> PostgreSQL
           |
    Solana deposit program
           |
    Program log subscription
           |
    Centralized relayer --> Zcash RPC --> Shielded payout

Demo mode: deposit router --> in-memory simulator --> database/status
```

[Architecture and known integration gaps](docs/ARCHITECTURE.md)

## Commands

| Command | Purpose / caveat |
| --- | --- |
| `bun run dev:web --env-mode=loose` | Next.js development server, port 3001; forwards local environment through Turbo |
| `bun run build` | Workspace build; does not build the Anchor program |
| `bun run check-types` | Runs type-check tasks declared by packages, not every file in the monorepo |
| `bun run check` | Biome formatting/linting **with writes** |
| `bun run db:push` | Applies schema to the configured database; inspect the target first |
| `bun run db:studio` | Opens Drizzle Studio |
| `bun run relayer` | Starts event listener and payout code; keep off for demo-only development |
| `bun run verify` | Runs the supplied deployment-check script; not a security audit |

## Repository map

```text
apps/web/          Next.js UI and tRPC HTTP route
packages/api/      Routers, services, demo simulation and relayer startup
packages/db/       Drizzle schema and PostgreSQL access
packages/solana/   Anchor program, IDL and TypeScript client
packages/zcash/    Zcash RPC client
packages/config/   Shared TypeScript configuration
scripts/           Supplied verification/address scripts
docs/              Source-reviewed setup, architecture and provenance
```

## Before attempting any real transfer

- Align wallet, RPC, program ID and token mint on one test network. The wallet provider is hardcoded to Devnet, while the deposit form contains a mainnet USDC mint.
- Review authentication and authorization. Several deposit and receipt operations use public procedures; session context accepts JSON from a cookie/header.
- Review relayer replay handling, amount units, failure recovery, payout confirmations and exchange-rate assumptions.
- Resolve the `ZCASH_RPC_PASSWORD` versus `ZCASH_RPC_PASS` split and other environment-loading details.
- Do not treat generated demo addresses, fake transaction IDs or fee fallbacks as valid on-chain results.
- Arrange independent security review before any production use.

See [deployment cautions](DEPLOYMENT.md). No real-fund deployment is recommended by this documentation.

## Source, credits and license status

This is a source snapshot imported from [dinxsh/zwap](https://github.com/dinxsh/zwap), with revised documentation and a fresh import history. The new Git contributor list describes this import, not original code authorship.

The upstream README labels the project **MIT**, but the captured tree has no standalone license text or copyright notice. That statement is preserved as upstream's stated license, not replaced with an invented copyright owner. See [provenance](docs/PROVENANCE.md) for the exact source revision and licensing caveat.
