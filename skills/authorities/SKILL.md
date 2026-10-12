---
name: authorities
description: >
  Read-only check of a Solana mint's mint, freeze, and update authorities
  and Token-2022 extensions (transfer fee, permanent delegate, metadata
  pointer, and others). Flags risky extensions. Used by the check skill.
---

# Authorities

Read the mint. Say who can mint, freeze, change the fee, seize tokens, or edit the name. Run this on the target mint, and again on the quote mint once pool-math has identified it. File the quote pass under "quote asset".

## Call

```bash
curl -s "$RPC_URL" -H 'content-type: application/json' -d '{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getAccountInfo",
  "params": ["<MINT>", {"encoding": "jsonParsed"}]
}'
```

Require `result.value.owner` to be `TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb` (Token-2022) or `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA` (SPL Token). Any other owner means this address is not a mint. Stop and say so.

## Fields

From `result.value.data.parsed.info`:

| Field | Meaning |
|---|---|
| `mintAuthority` | If a pubkey, that key can mint more. `null` means the key is cleared. |
| `freezeAuthority` | If a pubkey, that key can freeze accounts. `null` means cleared. |
| `supply` | Raw base units. Divide by `10^decimals`. |
| `decimals` | Use this for every amount of this mint. |
| `extensions[]` | Token-2022 only. Each item has `extension` and `state`. |

Extensions to read when present:

- `transferFeeConfig`: `newerTransferFee.transferFeeBasisPoints` (100 basis points = 1%), `maximumFee`, `transferFeeConfigAuthority`, `withdrawWithheldAuthority`, `withheldAmount`. Also read `olderTransferFee` and keep the one whose `epoch` is current if they differ. `getEpochInfo` gives the epoch if you need it; if both epochs match, either fee object is fine.
- `permanentDelegate`: `delegate`.
- `metadataPointer`: `authority`, `metadataAddress`.
- `tokenMetadata`: `updateAuthority`, `name`, `symbol`, `uri`. This is the on-mint metadata, which the metadata skill continues.
- `pausableConfig`: `authority`, `paused`.
- `transferHook`: `authority`, `programId`.
- `defaultAccountState`: `accountState`.
- `mintCloseAuthority`, `nonTransferable`, `interestBearingConfig`, `scaledUiAmountConfig`, `confidentialTransferMint`, `groupPointer`, `memberPointer`.

Also open the creation transaction later (creation-tx skill). If its inner instructions include `initializeTransferFeeConfig` or `setAuthority`, quote those authorities next to the current account. A difference means the key changed after creation. You do not have to find the change transaction; say it was not located.

## How to flag

Lead with anything in the first list. Phrase each as a signal.

**Call out in the first lines**

- `mintAuthority` is a pubkey. That key can increase supply.
- `freezeAuthority` is a pubkey. That key can freeze token accounts.
- `permanentDelegate.delegate` is set. That key can move tokens out of holders.
- `pausableConfig.paused` is true.
- `nonTransferable` is present.
- `transferHook.programId` is a pubkey (a program runs on transfer).
- `defaultAccountState.accountState` is `frozen`.

**Mention, without treating them as the same severity**

- Transfer fee basis points above 0. Name the percent (`basisPoints / 100`), the config authority (can change the rate), and the withdraw authority (can take withheld tokens).
- `maximumFee` so large that it does not cap a normal transfer (compare it to `supply`).
- `metadataPointer.authority` set: the pointer can be moved.
- `tokenMetadata.updateAuthority` set: name, symbol, and URI can change. The metadata skill covers the URI.
- `scaledUiAmountConfig`: the displayed amount and the raw amount differ. Pool math must use raw amounts.
- Quote mint only: say clearly that these keys belong to the quote asset, not the token under review. A permanent delegate on the quote mint is still relevant, because the pool's quote vault holds that asset.

Cleared authorities are a finding too: "`mintAuthority` is null", "`freezeAuthority` is null".

## Report lines

```text
**[verified]** Token program is Token-2022. Mint authority is null. Freeze authority is null. Supply is <ui> (raw <supply>, decimals <d>).
**[verified]** Transfer fee is <bps> basis points (<percent>%). Withdraw-withheld authority is `<addr>`. Fee-config authority is `<addr>`. Withheld on the mint right now: <ui> tokens.
**[verified]** No permanent-delegate extension on this mint.
**[heuristic]** The fee-config authority can change the rate later. That is a key-person signal on the tax, separate from whether mint and freeze are cleared.
```

Thresholds: there is no "safe percent". Any non-null mint, freeze, or permanent delegate is worth the first line of the summary. A transfer fee is worth a line whenever basis points are above 0.
