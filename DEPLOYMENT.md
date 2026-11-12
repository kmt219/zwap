# Deployment cautions

This snapshot is for code review and local development. It is not approved for production or real-fund use. No deployment was performed as part of the import.

Start with [local setup](docs/SETUP.md) and [architecture findings](docs/ARCHITECTURE.md). The original README referenced this file, but no such deployment guide was present in the captured tree; this document fills that missing link without claiming an operational deployment.

Before a testnet deployment, independently validate:

- One consistent network, deployed program ID, IDL/client and token mint across the wallet, browser, relayer and Anchor configuration.
- Database access, migrations, backups, environment loading and server-side session/authorization design.
- Solana amount units, event parsing, duplicate/reordered events, retry behavior and transaction confirmation.
- Zcash RPC version, testnet configuration, funding, address validation, asynchronous send status and wallet key custody.
- Relayer accounting and reconciliation between database state and both chains.
- Rate sourcing, liquidity, fees, slippage, refunds, failures and recovery paths.
- Secret management and logging. Never expose private RPC credentials or signing keys in browser-prefixed variables.
- Independent application/program review and a test matrix covering interrupted and duplicated operations.

A static frontend deployment alone does not provide the database, persistent relayer or Zcash node. The upstream `vercel.json` is configuration, not proof that all services can run there.

Do not run a mainnet deployment or transfer funds based solely on these docs.
