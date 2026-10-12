---
name: creation-tx
description: >
  Find and decode the Solana transaction that created a mint. Lists
  instructions, labeled programs, SOL and token transfers to other
  wallets, and the launch-tool hint from the metadata URI. Read-only.
---

# Creation transaction

Find the transaction that created the mint. List every outer instruction, the programs, and every SOL or token movement to a wallet that is not the fee payer.

## Find it

Stop at the first method that works.

1. If the user already passed a signature, `getTransaction` it and confirm the mint is created in that receipt (a `mintTo` or `initializeMint2` for this mint, or the mint account's pre-balance is 0 and post-balance is above 0). If the user did not pass one, continue.
2. LaunchLab pool. Quote mint comes from Dexscreener or from a CPMM pool (pool-math). Derive the pool PDA with seeds `["pool", <base-mint>, <quote-mint>]` and program `LanMV9sAd7wArD4vJFi2qDdfnVhFxYSUg6eADduJ3uj`. Your client's `findProgramAddress` is enough; bump search from 255 downward, SHA-256 of `seeds || bump || program_id || "ProgramDerivedAddress"`, first result that is not a valid ed25519 point. Then:

```bash
curl -s "$RPC_URL" -H 'content-type: application/json' -d '{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getSignaturesForAddress",
  "params": ["<LAUNCH_POOL>", {"limit": 1000}]
}'
```

Page with `"before": "<oldest signature>"` until a short page. The oldest signature is the candidate. Confirm with `getTransaction`.

3. If there is no LaunchLab pool, page `getSignaturesForAddress` on the mint itself, newest first, budget 20 pages of 1000. The creation receipt is the one where this mint is initialized. If the budget runs out, say the creation transaction was not found in 20,000 signatures.

```bash
curl -s "$RPC_URL" -H 'content-type: application/json' -d '{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getTransaction",
  "params": ["<SIGNATURE>", {
    "encoding": "jsonParsed",
    "maxSupportedTransactionVersion": 1
  }]
}'
```

## Fields

- `result.slot`, `result.blockTime` (Unix seconds, UTC), `result.meta.err`, `result.meta.fee`, `result.meta.computeUnitsConsumed`.
- `result.meta.logMessages` — lines containing `Instruction:` name the calls even when the outer instruction is raw.
- `result.transaction.message.accountKeys[]` — with `jsonParsed`, each entry is `{pubkey, signer, writable}`. Fee payer is the first signer (`accountKeys[0]`).
- `result.transaction.signatures` — length is the number of signatures. Base fee is `5000 * signatures` lamports (**derived**). `meta.fee` minus that base fee is the priority fee in lamports when the result is ≥ 0.
- Version 1 messages include `transactionConfig`: `computeUnitLimit`, `priorityFee` (lamports, when the client set one), `loadedAccountsDataSizeLimit`. Report those when present.
- Outer `message.instructions[]` and `meta.innerInstructions[]`.
- Parsed instructions: `parsed.type` and `parsed.info`.
- Raw instructions: `programId`, `accounts`, and the base58 `data` (you do not need to decode the data if the logs and inner parsed instructions already say what happened).
- `meta.preBalances` / `postBalances` line up with `accountKeys` by index. Delta lamports = post − pre.
- `meta.preTokenBalances` / `postTokenBalances`: match on `accountIndex` + `mint`. Use `uiTokenAmount.amount` (raw string) and `decimals`. `owner` is the wallet, not the token account.

## What to write

For each outer instruction, one line:

```text
#<n> <program label or address> — <parsed type or Instruction: name from logs>
```

Then a transfer table. Include an inner transfer when it moves SOL or tokens to an owner other than the fee payer:

| From | To | Asset | Amount | Where it sits |
|---|---|---|---|---|
| fee payer | … | SOL | lamports / 1e9 | outer instruction #n |

"Third party" means the owner is not the fee payer and not this mint's curve vault or pool vault. Name the owner. Rent that creates the mint or the fee payer's own token account is rent, not a payment to a third party; you can mention the rent total (`meta` account creations with preBalance 0) in one line so it is not confused with a tool fee.

Read `logMessages` for warnings such as `Mint has a permanent delegate`. The log does not say which mint. Check which mint was initialized in the inner instruction just above the log (often the quote vault). Attribute the warning to that mint.

## Report lines

```text
**[verified]** Creation signature `<sig>` in slot <slot> at <UTC> (<Europe/Rome>). Fee <fee> lamports. Error field is null.
**[verified]** Outer instructions: <numbered list>.
**[verified]** System transfer of <sol> SOL from the fee payer to `<dest>`.
**[verified]** Inner transferChecked of <amount> <symbol> from the fee payer's token account to `<owner>`'s account.
**[heuristic]** Program `<id>` is unlabeled. Logs say `Instruction: <name>`.
```

The same-slot neighbours of this transaction belong to the bundle-detection skill. You only have to record `slot` and the fee payer for that skill.
