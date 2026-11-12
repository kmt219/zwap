# Architecture and source-review findings

## Frontend and API

`apps/web` is a Next.js 16 / React 19 app. Its HTTP route exposes the routers from `packages/api`. Wallet adapters include Phantom, Solflare, Torus and Ledger. Deposit/status pages use tRPC and PostgreSQL rows from `packages/db`.

The deposit router has two branches:

- Demo: insert a row with a fake Solana signature and enqueue an in-process status simulator. Later updates write a synthetic Zcash transaction ID.
- Non-demo: create a pending record; the frontend constructs, signs and sends a Solana transaction, then records its signature.

## Solana and relayer

`packages/solana` contains the Anchor program, IDL and TypeScript client. The workspace `build` command only prints a note for this package; it does not compile the Rust program. Program compilation is a separate Anchor task.

The standalone relayer subscribes to program logs, parses deposit events, finds matching database rows and calls the Zcash client. This is a centralized service with operator trust, not an atomic cross-chain protocol.

## Source-review concerns

These are observations from this snapshot, not a full audit or confirmed runtime failures:

1. **Network mismatch:** wallet-provider source hardcodes Solana Devnet; the deposit form declares a mainnet USDC mint. Changing a single environment variable is not a complete network switch.
2. **Program configuration:** placeholder program IDs appear in examples and Anchor configuration. Deployment and IDL/client alignment are not verified.
3. **Replay/settlement logic:** the relayer skips an event when a row already has the same Solana signature. The frontend also records that signature, so ordering and replay behavior need tests before trusting payouts.
4. **Authentication/authorization:** session context parses client-supplied JSON; deposit updates and receipt generation use public procedures. Ownership and tamper resistance require review.
5. **Amounts and rates:** fixed conversion rates and token base-unit handling need independent checks. Live fee estimation is not live price conversion.
6. **Demo address mismatch:** generated `demo-zaddr-...` strings do not meet the deposit router's `z`/`u1` regular expression.
7. **Ephemeral state:** demo phase tracking and address mappings use in-memory maps.
8. **Environment split:** the relayer and API Zcash client use different password variable names; separate processes load configuration differently.
9. **Privacy claims:** a shielded destination, signed receipt or displayed health widget does not prove end-to-end privacy or settlement.

## Verification status

Completed for this import: static source review, repository inventory, documentation cross-checks and source provenance capture.

Not completed: dependency installation, compilation, type checks, linting, database setup, Anchor tests, wallet flows, RPC integration, deployment, financial transfers or security audit. Original upstream completion notes are historical claims and are not adopted as fresh verification.
