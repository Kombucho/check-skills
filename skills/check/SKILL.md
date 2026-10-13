---
name: check
description: >
  Read-only due diligence on a Solana token by mint address.
  Invoke as "check <CA>". Orchestrates the authorities, creation-tx,
  bundle-detection, pool-math, holders, fee-flows, lp-safety,
  trading-patterns, dev-history, and metadata skills and writes a
  markdown report plus JSON. Never signs or sends transactions.
---

# Check

Invoke as `check <CA>`, where `<CA>` is a Solana mint address.

You are the agent. You make the RPC calls yourself. The sub-skills under `skills/*/SKILL.md` are the procedure. There is no runner script.

## Read-only

Use only read methods: `getAccountInfo`, `getTransaction`, `getBlock`, `getSignaturesForAddress`, `getTokenLargestAccounts`, `getTokenAccountsByOwner`, `getMultipleAccounts`, and public HTTP GETs with no API key.

Do not send a transaction, sign, ask for a key, or describe how to build a bundle, a launch, or a trade. Write signals ("these receipts match this pattern"), not accusations.

```bash
export RPC_URL="${RPC_URL:-https://api.mainnet-beta.solana.com}"
```

On HTTP 429, wait and retry the same public endpoint. Do not invent a key. Every `getTransaction` and `getBlock` passes `"maxSupportedTransactionVersion": 1` (version 0 is not enough for version-1 transactions on this cluster).

Template:

```bash
curl -s "$RPC_URL" -H 'content-type: application/json' -d '{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getAccountInfo",
  "params": ["<MINT>", {"encoding": "jsonParsed"}]
}'
```

## Labels

Tag every finding with one of:

- **verified** — a field you read from a response in this run.
- **derived** — arithmetic you show, whose inputs are verified.
- **heuristic** — a threshold, a name from the label table, or a pattern. Say what would make it wrong.

If a call fails or you stop at a page budget, write **not evaluated** and why. Leave it empty rather than copying a number from a blog.

## Order

Run the sub-skills in this order. Pass addresses you have already verified (quote mint, creation signature, slot, pool, early wallets) into the later skills so you do not rediscover them.

1. `skills/metadata/SKILL.md` — needs the mint account from the first `getAccountInfo`. Do that call once and share it.
2. `skills/authorities/SKILL.md` — same mint account. After pool-math knows the quote mint, run authorities on the quote mint too and file it under "quote asset".
3. `skills/creation-tx/SKILL.md`
4. `skills/bundle-detection/SKILL.md`
5. `skills/pool-math/SKILL.md`
6. `skills/lp-safety/SKILL.md`
7. `skills/holders/SKILL.md`
8. `skills/fee-flows/SKILL.md`
9. `skills/trading-patterns/SKILL.md`
10. `skills/dev-history/SKILL.md`

Then write the report below.

## Label table

Use these names when an address matches. The match is a label (heuristic), the account's presence in a receipt is verified.

| Address | Label |
|---|---|
| `11111111111111111111111111111111` | System Program |
| `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA` | SPL Token |
| `TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb` | Token-2022 |
| `ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL` | Associated Token Account |
| `ComputeBudget111111111111111111111111111111` | Compute Budget |
| `LanMV9sAd7wArD4vJFi2qDdfnVhFxYSUg6eADduJ3uj` | Raydium LaunchLab |
| `CPMMoo8L3F4NbTegBCKVNunggL7H1ZpdTHKxQB5qKP1C` | Raydium CPMM |
| `CAMMCzo5YL8w4VFF8KVHrK22GGUsp5VTaW7grrKgrWqK` | Raydium CLMM |
| `675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8` | Raydium AMM v4 |
| `6EF8rrecthR5Dkzon8Nwu78hRvfCKubJ14M5uBEwF6P` | pump.fun |
| `LBUZKhRxPF3XUpBCjp4YzTKgLccjZhTSDM9YuVaPwxo` | Meteora DLMM |
| `cpamdpZCGKUy5JxQXB4dcpGPiikHawvSWAd6mEn1sGG` | Meteora DAMM v2 |
| `Eo7WjKq67rjJQSZxS6z3YkapzY3eMj6Xy8X5EQVn5UaB` | Meteora Dynamic AMM |
| `whirLbMiicVdio4qvUfM5KAg6Ct8VwpYzGff3uctyCc` | Orca Whirlpool |
| `JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4` | Jupiter v6 |
| `MemoSq4gqABAXKb96qnH8TysNcWxMyWCqXgDLGmfcHr` | Memo |
| `6BwHHDg3u1854jC8PDLXvR4spTcLNaoBxLJNGC4nTESt` | StonkFun LaunchLab platform config (reward) |
| `5CEbueQnq1Ym2uSSx2xXds3jQAqT1BDnkA59RZobSPAG` | StonkFun platform wallet (label) |
| `5KXDF6QnqhBj72hDtJNkkpFaQVUfbFXNybMsp3DiK6tD` | StonkFun withdraw-authority wallet (label) |

An unknown program stays "unlabeled program `<address>`". If its logs contain `Instruction: SomeName`, you may quote that log. You may note a shared prefix with a metadata host (see the metadata skill). Do not upgrade a prefix into a product claim beyond that.

## Report

Write markdown the reader can follow, then a JSON object with the same findings.

```markdown
# Check: <name> (<symbol>)

**Mint:** `<CA>`
**When:** <UTC time of the run> (<Europe/Rome time>)
**RPC:** `<RPC_URL>`

## Summary

<One short paragraph: authorities, creation-slot pattern, block-0 share, exit if you measured one, pool, and the signal score. Name anything you left unevaluated.>

## Signal score

<From bundle-detection. Table of points. Band from that skill.>

## Authorities
## Metadata
## Creation transaction
## Same-slot activity
## Pool
## LP
## Holders
## Fee path
## Trading patterns
## Deployer

## Not evaluated

<Bullet list. This is required when any check stopped early.>
```

Each bullet in the sections starts with the label: `**[verified]**`, `**[derived]**`, or `**[heuristic]**`.

JSON shape:

```json
{
  "mint": "",
  "name": "",
  "symbol": "",
  "generated_at": "",
  "rpc": "",
  "summary": "",
  "signal_score": { "points": 0, "evaluated_max": 0, "defined_max": 110, "band": "", "unevaluated": [] },
  "findings": [{ "check": "", "level": "verified", "text": "" }]
}
```

Times: store UTC. When you show a clock time to a person, also show Europe/Rome with the abbreviation (CEST or CET).

## Several mints

Repeat this skill once per mint. Sort the summaries by signal-score points, high to low. Skip any trending-site API. No key, no login. v0.1 does not ship a sample table.
