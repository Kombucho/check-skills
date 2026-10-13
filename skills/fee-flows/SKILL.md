---
name: fee-flows
description: >
  Read-only trace of a Token-2022 transfer fee: withheld tokens, the
  withdraw authority, the swap that follows, and any later payout split.
  Generic for any fee mint. Used by the check skill.
---

# Fee flows

Follow withheld transfer fees from the mint to whoever receives the proceeds. This is generic. A StonkFun-style path (withdraw, swap, then a split of the quote token) is one pattern the receipts might show. Report the path you see.

If the mint has no `transferFeeConfig`, write that and stop.

## Calls

From the authorities read you already have `withdrawWithheldAuthority`, `transferFeeConfigAuthority`, `transferFeeBasisPoints`, and `withheldAmount`.

```bash
curl -s "$RPC_URL" -H 'content-type: application/json' -d '{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getTokenAccountsByOwner",
  "params": ["<WITHDRAW_AUTHORITY>", {"mint": "<MINT>"}, {"encoding": "jsonParsed"}]
}'
```

Then the recent signatures of that token account (the fee wallet's ATA for this mint):

```bash
curl -s "$RPC_URL" -H 'content-type: application/json' -d '{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getSignaturesForAddress",
  "params": ["<FEE_ATA>", {"limit": 20}]
}'
```

`getTransaction` on the newest 6 successful signatures. You are looking for a pair close in time:

1. `withdrawWithheldTokensFromMint` (or `FromAccounts`), authority = the withdraw authority, amount = the positive token delta on their ATA.
2. A later transaction, same fee payer, where that ATA's balance of this mint falls and some other mint (the quote) rises. The program id and the log `Instruction:` name tell you the venue (CPMM `SwapBaseInput`, LaunchLab `SellExactIn`, Jupiter, and so on).

Read `postTokenBalances` minus `preTokenBalances` for every owner. The pool side of a swap often receives **less** than the authority lost, by about the transfer-fee rate. Compute `lost - received` and compare to `lost * basisPoints / 10000`. If it matches, the fee applied to the authority's own swap. **Derived.**

## After the swap

The quote tokens land in an ATA owned by the withdraw authority. `getSignaturesForAddress` on that quote ATA, `limit` 30. Find the next receipt where that owner's quote balance **falls** and the instruction is a `transfer` or `transferChecked` rather than another swap.

For each destination owner, record the raw amount. If two destinations recur across at least 3 such receipts and the ratio is stable within 0.5 percentage points, report:

```text
**[derived]** In <n> split receipts, destination A received <pct>% and destination B received <pct>% of the quote sent (sample, not a lifetime total).
```

If the next receipts are more swaps and you never see a decrease that is a transfer, say the split was not in the sample. Do not fill in a ratio from a write-up.

Name a destination with the label table when it matches. Otherwise print the address.

Also note, as its own line, whether `transferFeeConfigAuthority` and `withdrawWithheldAuthority` are the same key. One key that can both change the rate and withdraw is a key-person signal (**heuristic**).

## Report lines

```text
**[verified]** `<sig>` at <UTC>: `<authority>` signed withdrawWithheldTokensFromMint for <amount> <symbol> into `<ata>`.
**[verified]** `<sig>` one second later: the same signer swapped that balance on Raydium CPMM. The pool's <symbol> balance rose by <received>; the difference is <fee> (<pct>%).
**[verified]** The quote proceeds in that receipt stayed with `<authority>`. No outbound quote transfer was in this sample of <n> signatures.
```

Lifetime totals are out of scope. Say the sample size.
