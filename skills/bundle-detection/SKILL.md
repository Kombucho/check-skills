---
name: bundle-detection
description: >
  Read-only signals around a Solana token's creation slot: co-signed
  funding, same-slot buys, block-0 supply share, and how fast early
  wallets sold. Outputs a signal score. Does not accuse and does not
  explain how to build a bundle.
---

# Bundle detection

Look at the creation slot and the slots next to it. Describe what the receipts show. A co-signed transaction is a fact about signatures. It is a signal, not a finding that someone committed a crime.

## Calls

Creation slot `S` and signature come from the creation-tx skill.

```bash
curl -s "$RPC_URL" -H 'content-type: application/json' -d '{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getBlock",
  "params": [<S>, {
    "encoding": "json",
    "transactionDetails": "signatures",
    "rewards": false,
    "maxSupportedTransactionVersion": 1
  }]
}'
```

Repeat for `S-1` and `S+1`. Note `blockTime` and the index of the creation signature.

Prefer a second `getBlock` with `"transactionDetails": "accounts"` on slot `S` (and the neighbours if the response is small enough to handle). Each transaction lists its account keys. Keep every signature whose keys include the mint or the fee payer. Fetch those with `getTransaction`. If the node rejects `accounts`, fetch the five signatures before the creation signature and the five after it, plus any you already know, and write that the rest of the slot was not decoded.

For each kept transaction, read the same fields as the creation-tx skill: signers, system transfers, token balance deltas, logs, `meta.fee`, `transactionConfig`.

## Patterns that add points

Score only what you actually decoded. Points are additive. Cap the reported total at the sum of the rows you evaluated.

| Signal | Points | You need |
|---|---:|---|
| A wallet other than the creation fee payer gains this mint in slot `S` | 25 | a token balance delta in a fetched receipt |
| A transaction in slot `S` is signed by the fee payer and at least one of those wallets, and moves SOL from them to the fee payer | 25 | `signer: true` on each, and a system transfer |
| The creation transaction itself sends SOL or the quote token to those same wallets | 15 | inner or outer transfer in the creation receipt |
| At the end of slot `S`, fee payer + those wallets hold ≥ 20% of the minted supply, using post-fee balances | 15 | last receipt in the slot that touches each wallet |
| That same share is ≥ 30% | 10 | the same balances |
| Each of those wallets' balance of this mint later reaches 0 within 120 seconds of `blockTime` | 10 | their token-account history |
| The same co-signers, or the same lamport amounts, appear on another mint | 10 | a second mint you actually opened |

Defined maximum is 110. If a row is not evaluated, leave its points at 0 and list it under unevaluated. Do not award the last row from memory.

Bands, based on points that fired:

| Points | Band |
|---:|---|
| 0–24 | few of these signals |
| 25–49 | some of these signals |
| 50–74 | many of these signals |
| 75–110 | creation-slot cluster and a fast exit both showed up, or the score is in this range for another mix |

## Block-0 share

Minted supply is the `mintTo` amount in the creation receipt, not today's `supply` (tokens may have been burned since).

For each wallet, the spendable balance is the `postTokenBalances` raw `amount` on the last transaction you have for them in slot `S`. Token-2022 withholding means this is lower than the `transferChecked` amount. Report both:

- **verified** gross `transferChecked` amount
- **verified** post balance
- **derived** post balance / minted supply

Add the fee payer's post balance to the other wallets' post balances for the share used in the score.

## Later sells and SOL

For each of those wallets:

```bash
curl -s "$RPC_URL" -H 'content-type: application/json' -d '{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getTokenAccountsByOwner",
  "params": ["<WALLET>", {"mint": "<MINT>"}, {"encoding": "jsonParsed"}]
}'
```

If the account is closed, `getSignaturesForAddress` on the token account address you saw created in the buy receipt still returns history on an archival node.

Page until you pass `blockTime + 180` seconds, budget 5 pages. Fetch the receipts in that window where this mint's raw balance changes.

- Time to empty: `blockTime` of the first receipt whose post balance for that wallet is 0, minus the creation `blockTime`.
- Net SOL: that wallet's `postBalances` lamports on the receipt where the mint balance first hits 0, minus its `preBalances` lamports on the funding transaction (the one just before creation, or the earliest receipt in the slot where it sends SOL). Divide by 1e9. **Derived.** If the wallet key is missing from the later receipt, do not treat the balance as zero; say the SOL figure is missing.
- Count receipts where the mint balance falls. A transfer to another of the same wallets is a fall. Say so, and name the signature, so a pass-between is not described as a sale to the pool.
- Pre-balances on the funding transaction (lamports / 1e9) answer "were these wallets empty". Report them. Empty-or-not is a signal only together with the co-sign; a large balance is not proof of anything by itself.

## Other launches

Best effort, separate from the score unless you finish it. Take one of the co-signing wallets and:

```bash
curl -s "$RPC_URL" -H 'content-type: application/json' -d '{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getSignaturesForAddress",
  "params": ["<WALLET>", {"before": "<FUNDING_SIG>", "limit": 40}]
}'
```

Fetch at most 40 of those transactions. Count how many are signed by the same set of wallets or contain the same system-transfer lamport amounts (for example 2.15 SOL, 1.81 SOL, 1.94 SOL if those were the amounts here). If you hit the budget, report the count you saw and mark the rest not evaluated. Do not extrapolate.

## Report lines

```text
**[verified]** Slot <S> has <n> signatures. Creation is index <i>. The previous signature is `<fund>` and the next is `<buy>`.
**[verified]** `<fund>` is signed by <list>. System transfers: <wallet> sent <sol> SOL to the fee payer.
**[derived]** End of slot <S>, spendable balances sum to <tokens> of <minted>, which is <pct>% .
**[derived]** Signal score <points> / <evaluated max> evaluated (<defined> defined). Band: <band>. Points: <one line each>.
```

Write the score as a sum of signals. Do not write that the wallets "are a team" or "rugged". "These addresses signed one transaction together" is the sentence.
