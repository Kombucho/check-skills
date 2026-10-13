---
name: holders
description: >
  Read-only top-holder concentration for a Solana mint, excluding pool
  vaults and burn addresses, plus a best-effort cluster by first SOL
  funding source. Used by the check skill.
---

# Holders

Who holds the supply now, outside the pool and the burn addresses, and whether those wallets' first SOL came from one place.

## Calls

```bash
curl -s "$RPC_URL" -H 'content-type: application/json' -d '{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getTokenLargestAccounts",
  "params": ["<MINT>"]
}'
```

The method returns at most 20 token accounts. Say that the share is "of the top 20", not of every holder.

Resolve owners in batches of 8:

```bash
curl -s "$RPC_URL" -H 'content-type: application/json' -d '{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getMultipleAccounts",
  "params": [["<ATA>", "<ATA>"], {"encoding": "jsonParsed"}]
}'
```

Read `data.parsed.info.owner` and `tokenAmount.amount`.

Exclude from the "holder" total, and list them separately:

- The pool vaults you found in pool-math (and the LaunchLab curve vault, if it still holds tokens).
- Burn addresses `1nc1nerator11111111111111111111111111111111` and any token account whose owner is the System Program **and** whose entire balance is the well-known incinerator. A system-owned wallet that merely holds a lot is still a holder; do not call it burned unless you also show a burn instruction or a known burn address.

Supply is the mint's current `supply` from the authorities read (same session). Percent = raw amount / supply.

## Thresholds (heuristic)

Apply to the sum of the largest non-excluded holders, capped at the 10 largest of those:

| Top 10 share of supply | Line to write |
|---:|---|
| ≥ 50% | High concentration signal among the top 10 non-pool holders. |
| ≥ 30% and < 50% | Elevated concentration signal. |
| < 30% | Top 10 are under the 30% line. Still list the top 5. |

The creation-slot share from bundle-detection is a different number. Keep it in that section. This section is the current snapshot.

## Common funding source

For each of the top 5 non-excluded owners, budget 3 pages:

```bash
curl -s "$RPC_URL" -H 'content-type: application/json' -d '{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getSignaturesForAddress",
  "params": ["<OWNER>", {"limit": 1000}]
}'
```

Page with `before` until the short page or the budget ends. The oldest signature is the candidate "first activity". `getTransaction` it and read the first positive lamport delta for that owner (post − pre on their account index). The counterparty is the funding source if a system transfer's destination is this owner.

**Heuristic:** three or more of the five share the same funding source. Say "common first-SOL source `<addr>` for `<n>` of the top 5 examined". If the page budget expires, say the first inflow was not reached. Do not guess the source from a later transaction.

## Report lines

```text
**[verified]** Largest token account `<ata>` holds <ui> (raw <amount>), owner `<wallet>`.
**[derived]** Top 10 non-pool holders in this list hold <pct>% of supply <supply_ui>. That is under the 30% line.
**[heuristic]** First-SOL source was not reached within 3 pages for <wallets>. Not evaluated.
```
