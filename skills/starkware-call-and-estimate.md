---
name: Call a contract and estimate fees before submitting a transaction
description: Read contract state with starknet_call, fetch the account nonce, estimate the fee, then submit a signed invoke transaction and poll its status.
api: openrpc/starkware-starknet_api_openrpc.json
operations: [starknet_call, starknet_getNonce, starknet_estimateFee, starknet_addInvokeTransaction, starknet_getTransactionStatus]
method: generated
---

# Call a contract and estimate fees, then invoke

Read-only calls and fee estimation live in the read API
(`openrpc/starkware-starknet_api_openrpc.json`); the write operation
`starknet_addInvokeTransaction` lives in the write API
(`openrpc/starkware-starknet_write_api.json`).

## Steps

1. **Read state** — call `starknet_call` with `{ contract_address, entry_point_selector,
   calldata }` and a `block_id` to read a view function. A bad selector returns
   `21 ENTRYPOINT_NOT_FOUND`; a missing contract returns `20 CONTRACT_NOT_FOUND`;
   execution reverts surface as `40 CONTRACT_ERROR`.
2. **Get the account nonce** — call `starknet_getNonce` with `block_id` and the
   account `contract_address`. Starknet has no idempotency key: the nonce is the
   replay-protection mechanism (see `conventions/starkware-conventions.yml`).
3. **Estimate the fee** — call `starknet_estimateFee` with the broadcasted
   transaction(s) and a `block_id`. Use the result to set resource bounds.
4. **Submit the invoke** — sign the transaction with the account key and call
   `starknet_addInvokeTransaction` (write API) with the broadcasted invoke v3
   transaction. It returns a `transaction_hash`. Watch for `52 INVALID_TRANSACTION_NONCE`,
   `54 INSUFFICIENT_ACCOUNT_BALANCE`, `55 VALIDATION_FAILURE`, or
   `59 DUPLICATE_TX`.
5. **Poll status** — call `starknet_getTransactionStatus` with the returned
   `transaction_hash` until it reaches an accepted status.

## Rules

- Transactions are authorized by the account signature + nonce, not an API
  credential — see `authentication/starkware-authentication.yml`.
- All errors are JSON-RPC error objects — see `errors/starkware-problem-types.yml`.
