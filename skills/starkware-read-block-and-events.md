---
name: Read Starknet blocks, transactions, and events over JSON-RPC
description: Query the Starknet Node JSON-RPC API for the latest block, a transaction receipt, and filtered events with pagination.
api: openrpc/starkware-starknet_api_openrpc.json
operations: [starknet_blockNumber, starknet_getBlockWithTxs, starknet_getTransactionReceipt, starknet_getEvents]
method: generated
---

# Read Starknet blocks, transactions, and events

Use the Starknet Node JSON-RPC API (spec 0.10.4-rc.0) against any full-node
endpoint (`STARKNET_RPC_URL`). All calls are JSON-RPC 2.0 POST bodies; no API
auth is defined by the spec (commercial providers may require a key in the URL
or `x-api-key` header — see `authentication/starkware-authentication.yml`).

## Steps

1. **Get the chain tip** — call `starknet_blockNumber` (no params) to get the
   latest block number. `block_id` in later calls accepts `{ "block_number": N }`,
   `{ "block_hash": "0x..." }`, or a tag (`"latest"`, `"pre_confirmed"`).
2. **Read a block with transactions** — call `starknet_getBlockWithTxs` with
   `block_id`. Returns the block header plus the full `transactions` list.
3. **Fetch a receipt** — call `starknet_getTransactionReceipt` with
   `transaction_hash`. Handle error `29 TXN_HASH_NOT_FOUND` if the hash is unknown.
4. **Query events with pagination** — call `starknet_getEvents` with a `filter`
   of `{ from_block, to_block, address?, keys?, chunk_size, continuation_token? }`.
   The response returns `events` plus a `continuation_token`; pass it back on the
   next call until it is absent. Keep `chunk_size` within limits (error
   `31 PAGE_SIZE_TOO_BIG`) and `keys` within the filter cap (error
   `34 TOO_MANY_KEYS_IN_FILTER`); a stale cursor yields
   `33 INVALID_CONTINUATION_TOKEN`.

## Rules

- Errors are JSON-RPC error objects `{ code, message, data? }` — see
  `errors/starkware-problem-types.yml`.
- Pagination is cursor-based on `continuation_token` only — see
  `conventions/starkware-conventions.yml`.
