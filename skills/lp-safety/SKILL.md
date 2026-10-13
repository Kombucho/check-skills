---
name: lp-safety
description: >
  Read-only check of whether a Solana pool's LP is burned or locked,
  who the pool creator is, and who is recorded as the creator-fee
  recipient. Used by the check skill.
---

# LP safety

Can the liquidity be withdrawn, and who is recorded as receiving the creator fee.

Use the pool account you already decoded in pool-math. This skill is written for Raydium CPMM. For another program, say the layout was not applied and report only what `getTokenLargestAccounts` shows for the LP mint if you have one.

## Calls

LP mint and `lp_supply` come from the pool bytes. Mint supply:

```bash
curl -s "$RPC_URL" -H 'content-type: application/json' -d '{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getAccountInfo",
  "params": ["<LP_MINT>", {"encoding": "jsonParsed"}]
}'
```

`info.supply` is the circulating LP mint supply (raw). `lp_supply` on the pool is the pool's own counter. A large gap (`lp_supply` well above mint `supply`) means LP was destroyed and the pool counter was not reduced by the same amount. The burned fraction from that gap is:

```text
burned_by_supply = max(lp_supply - 1, mint_supply) - mint_supply
fraction = burned_by_supply / max(lp_supply - 1, mint_supply)
```

Mark it derived. A gap of a few raw units is dust, not a lock.

Holders:

```bash
curl -s "$RPC_URL" -H 'content-type: application/json' -d '{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getTokenLargestAccounts",
  "params": ["<LP_MINT>"]
}'
```

Resolve the top accounts' owners with `getMultipleAccounts` (jsonParsed), the same way as the holders skill. Then `getAccountInfo` on the largest owner:

- Owner is `1nc1nerator11111111111111111111111111111111`: **verified** burned to the incinerator.
- Owner's account is owned by a program (not the System Program) and you see no withdraw path in the program's logs: say which program, and call the lock **heuristic** unless you decoded a "no withdraw" instruction.
- Owner is a system account (`owner` of that account is the System Program, `space` 0): it is a keypair address holding LP. Report the share. Do **not** call it burned. A public API `burnPercent` that matches this share can be quoted as an API figure (**heuristic** / from that HTTP response) next to the on-chain share.

Sample one recent signature of the largest LP token account. If the logs are `SampleMint` or another read, and the LP raw balance does not change, say the recent signatures you saw did not move the LP. That is not proof it cannot move.

`pool_creator` on the pool account is the creator-fee recipient field for CPMM when `creator_fee_rate` is non-zero. **Verified** address, **derived** role from the layout. Confirming a fee actually arrived requires a transfer you decoded; if you did not decode one, say the recipient is the field on the pool, and that no payout receipt was opened.

`protocol_owner` and `fund_owner` on the amm config are the other fee recipients. Quote them.

## Thresholds

| Observation | Line |
|---|---|
| Mint supply is under 1% of `lp_supply` | LP supply was destroyed. Withdrawal of that share is not available. |
| ≥ 95% of LP mint supply sits at the incinerator or in a program-owned locker you identified | Locked or burned share is ≥ 95%. |
| ≥ 95% sits in a system account | Concentrated in one wallet. Not a proof of a lock. |
| Other wallets hold a visible remainder | Those holders can withdraw their fraction. Give the fraction. |

## Report lines

```text
**[verified]** LP mint `<mint>` supply <supply>. Pool lp_supply <lp_supply>. Difference <n> raw units.
**[verified]** Largest LP account `<ata>` holds <ui>, owner `<wallet>`. That owner's account is system-owned, space 0.
**[derived]** That balance is <pct>% of the LP mint supply.
**[heuristic]** Raydium's public pool API burnPercent is <pct>, in line with that share. This run did not decode a burn or a lock instruction.
**[verified]** pool_creator is `<addr>`. creator_fee_rate on the config is <n> / 1_000_000.
```
