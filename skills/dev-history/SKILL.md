---
name: dev-history
description: >
  Best-effort read-only look at a Solana token deployer's and funder's
  other launches and what happened to them. Used by the check skill.
---

# Deployer history

Name the fee payer on the creation transaction and, if you have the budget, see whether that wallet has created other mints.

This check is best effort. A page limit is a normal outcome. Write "not evaluated" when you stop. Do not estimate a count you did not count.

## Who the deployer is

**Verified:** the creation transaction's first signer (fee payer), from `accountKeys[0]`, plus that account's `preBalances` lamports on the creation transaction or on the funding transaction immediately before it. That balance is "SOL in the wallet at that receipt", not a lifetime total.

If bundle-detection found a funding transaction with extra signers, those signers are the "funder" set. List them. Their pre-balances belong in the bundle section; you can refer to them here instead of repeating the table.

## Other launches

Budget: 20 transactions, or 2 pages of signatures, whichever you hit first.

```bash
curl -s "$RPC_URL" -H 'content-type: application/json' -d '{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getSignaturesForAddress",
  "params": ["<FEE_PAYER>", {"before": "<CREATION_SIG>", "limit": 20}]
}'
```

`getTransaction` each. Count a transaction as another launch only when its logs or parsed instructions show `InitializeWithToken2022`, `InitializeMint`, or `initializeMint2` for a **different** mint, or a pump.fun / LaunchLab / Meteora initialize you can name from the label table.

For each such mint, one follow-up only:

- `getAccountInfo` on that mint (authorities skill, short form): mint authority, freeze authority, transfer fee.
- If Dexscreener or Raydium returns a pool, record its quote reserve in one line (pool-math, abbreviated). You do not need a full second report.

Outcome words you may use, each tied to a receipt:

- "mint authority is null" (verified on that mint)
- "a pool account exists, quote reserve `<amount>`" (verified)
- "same co-signers as this launch" (verified if the signers match)

Do not write "rugged" or "abandoned". A dust reserve is a small reserve; say the number.

If the 20 transactions contain no other initialize, say that. The wallet can still have older launches outside the budget.

## Report lines

```text
**[verified]** Creation fee payer is `<addr>`. On the funding receipt immediately before creation, that account's pre-balance is <sol> SOL.
**[verified]** The 20 signatures before the creation signature include the funding transaction and no other mint initialize. Older history was not paged.
```

Or, when you do find one:

```text
**[verified]** `<sig>` initializes mint `<other>`. Mint authority on that mint is <state>.
```
