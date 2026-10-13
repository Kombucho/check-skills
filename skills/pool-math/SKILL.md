---
name: pool-math
description: >
  Find a Solana token's main pools and compute price as quote reserve
  over token reserve, liquidity, and the CPMM constant-product fee.
  Separates vault balances from reserves by subtracting uncollected fees.
  Read-only.
---

# Pool math

Find the main pool, compute price from reserves, and show what the fee does to a small swap. Vault balances include uncollected fees. Reserves do not.

## Find candidate pools

Hints (confirm on chain before you quote a number):

```bash
curl -sS -m 20 "https://api.dexscreener.com/latest/dex/tokens/<MINT>"
curl -sS -m 20 "https://api-v3.raydium.io/pools/info/mint?mint1=<MINT>&poolType=all&poolSortField=liquidity&sortType=desc&pageSize=5&page=1"
```

For each candidate address:

```bash
curl -s "$RPC_URL" -H 'content-type: application/json' -d '{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getAccountInfo",
  "params": ["<POOL>", {"encoding": "base64"}]
}'
```

Keep pools whose `value.owner` is a program you can decode. The main pool is the one with the largest quote-side reserve after the math below. Name the others in one line.

LaunchLab curve accounts are not CPMM pools. If the token is still on a curve, say the spot price is the curve's, and that this skill's formula is for a constant-product pool. Do not force the CPMM formula onto a curve.

## Raydium CPMM layout

Owner must be `CPMMoo8L3F4NbTegBCKVNunggL7H1ZpdTHKxQB5qKP1C`. Data length 637. Skip the 8-byte discriminator. Then:

| Offset | Field |
|---:|---|
| 8 | `amm_config` pubkey |
| 40 | `pool_creator` pubkey |
| 72 | `token_0_vault` pubkey |
| 104 | `token_1_vault` pubkey |
| 136 | `lp_mint` pubkey |
| 168 | `token_0_mint` pubkey |
| 200 | `token_1_mint` pubkey |
| 232 | `token_0_program` pubkey |
| 264 | `token_1_program` pubkey |
| 296 | `observation_key` pubkey |
| 328 | five u8: `auth_bump`, `status`, `lp_decimals`, `mint_0_decimals`, `mint_1_decimals` |
| 333 | seven u64 little-endian: `lp_supply`, `protocol_fees_token_0`, `protocol_fees_token_1`, `fund_fees_token_0`, `fund_fees_token_1`, `open_time`, `recent_epoch` |

`open_time` is a Unix timestamp stored on the account.

Amm config (owner the same CPMM program, typically 236 bytes). After the 8-byte discriminator: `bump` u8, `disable_create_pool` u8, `index` u16, then four u64s `trade_fee_rate`, `protocol_fee_rate`, `fund_fee_rate`, `create_pool_fee`, then `protocol_owner` and `fund_owner` pubkeys. The next u64 is `creator_fee_rate` when it matches Raydium's public pool API field of the same name. Record the following u64 as well and leave it unnamed if you cannot match it to a documented field.

Fee denominator is `1_000_000`.

```text
trade_pct = trade_fee_rate / 1_000_000
creator_pct = creator_fee_rate / 1_000_000
total_fee_rate = trade_fee_rate + creator_fee_rate
```

`protocol_fee_rate` and `fund_fee_rate` are shares of the **trade** fee, same denominator (so 120000 means 12% of the trade fee, not 12% of the swap). Mark that sentence derived from the CPMM layout.

## Vault versus reserve

```bash
curl -s "$RPC_URL" -H 'content-type: application/json' -d '{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getAccountInfo",
  "params": ["<VAULT>", {"encoding": "jsonParsed"}]
}'
```

Use `tokenAmount.amount` (raw) and the mint's `decimals`. Ignore `uiAmountString` when the mint has `scaledUiAmountConfig`: that string is raw / 10^decimals times the active multiplier. Price for the curve uses the raw figure. You may also show a displayed price (raw price times the multiplier) and label it derived.

```text
reserve_raw = vault_raw - protocol_fees - fund_fees
```

If a newer pool layout adds another fee bucket, subtract that too. If `reserve_raw` would go negative, stop and say the layout did not match.

```text
price = (reserve_quote_raw / 10^quote_decimals) / (reserve_token_raw / 10^token_decimals)
```

Price is quote tokens per one target token. Liquidity is the two reserve amounts. Also show the two vault amounts beside them so the fee gap is visible.

Repeat both vault reads and the pool read in the same pass and record the context slot. Fees move.

## Fee impact

For a hypothetical input of `1` quote token (raw = `10^quote_decimals`), exact-in:

```text
fee = ceil(amount_in * total_fee_rate / 1_000_000)
amount_in_after_fee = amount_in - fee
amount_out = floor(reserve_out * amount_in_after_fee / (reserve_in + amount_in_after_fee))
```

Also compute `amount_out` with `fee = 0`. The gap is the fee's effect on this size, **derived**. Say the size (1 quote token) so nobody treats it as the fee on every trade.

The Token-2022 transfer fee is a different mechanism. If the target mint has one, say so in the same section and do not add the percents together. On a sell, withholding happens when tokens move into the pool; the CPMM fee is what the pool program takes from the swap. Quote both rates and leave the combination as a heuristic if you did not decode a live swap.

## Report lines

```text
**[verified]** Pool `<pool>` is owned by Raydium CPMM. token_0 is <quote>, token_1 is <symbol>. pool_creator is `<addr>`. open_time is <UTC>.
**[verified]** trade_fee_rate <n>, creator_fee_rate <n>, protocol_fee_rate <n>, fund_fee_rate <n> on config `<config>`.
**[derived]** Total CPMM fee rate is <total>/1_000_000 = <pct>%.
**[derived]** Quote reserve <q> (vault <qv> minus protocol <p> minus fund <f>). Token reserve <t>. Price = <q>/<t> = <price> quote per token.
**[derived]** A 1 <quote> exact-in swap at these reserves yields <out> tokens after the fee, against <out0> with a zero fee.
**[heuristic]** Raydium's public API `feeRate` is <x>, which matches the sum above. The API is a hint; the rates above are from the account bytes.
```
