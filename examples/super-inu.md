# Check: Super Inu (SI)

**Mint:** `DEW9dSN6QpWyNthphCpMmAbZP1Q4cEKR9xQXAri98WDP`
**When:** 12 Oct 2026, reads through context slot 456191786 (21:12 UTC receipts are 23:12 CEST)
**RPC:** `https://api.mainnet-beta.solana.com` (a few `getTokenLargestAccounts` calls were repeated on `https://public.rpc.solanavibestation.com` after HTTP 429)

This report was produced by following `skills/check/SKILL.md` and the ten sub-skills against live mainnet. No transaction was signed or sent.

## Summary

Super Inu is a Token-2022 mint. Mint authority and freeze authority are null. A 1% transfer fee is set, and the withdraw-withheld authority is `5KXDF6QnqhBj72hDtJNkkpFaQVUfbFXNybMsp3DiK6tD`. The creation transaction in slot 449099389 (21 Sep 2026, 15:44:55 UTC, 17:44:55 CEST) pays 0.07 SOL to a `rapid…` account and sends NVDAx to three wallets. The signature immediately before it is a four-signer funding transaction (the fee payer plus those three). The signature immediately after it is those three wallets buying SI. At the end of the slot their spendable balances plus the fee payer's spendable balance are 31.57% of the 1,000,000,000 minted. Those three balances are 0 by 15:46:11 UTC, 76 seconds later, and the SOL change from just before the funding transaction to that point is about +40.17 SOL combined. The graduated pool is Raydium CPMM `B4VFURUHHzyt8YzBGBV9jiarBvjh1EbMAbRNBnNqaxUD`. The signal score on the checks that were run is 100. Repeat launches, holder funding sources, and the quote-token split after the fee swap were not evaluated.

## Signal score

100 points fired, out of 100 evaluated (110 defined). Band: creation-slot cluster and a fast exit both showed up.

| Signal | Points |
|---|---:|
| Other wallets gained SI in the creation slot | 25 |
| Same-slot transaction signed by the fee payer and those wallets, SOL moving to the fee payer | 25 |
| Creation transaction sends the quote token to those wallets | 15 |
| End-of-slot spendable share ≥ 20% of minted supply | 15 |
| That share ≥ 30% (31.57%) | 10 |
| Those wallets' SI balance hit 0 within 120 seconds | 10 |
| Same co-signers or lamport amounts on another mint | not evaluated |

## Authorities

**[verified]** The mint account is owned by Token-2022 (`TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb`). `mintAuthority` is null. `freezeAuthority` is null. Decimals are 6. At context slot 456191784, `supply` is `996024042883960` (996,024,042.883960 SI).

**[verified]** The creation receipt mints `1000000000000000` raw units (1,000,000,000 SI) and then `setAuthority` with `authorityType` `mintTokens` and `newAuthority` null, in the same transaction. Today's supply is lower than the amount minted; this run did not locate a burn transaction.

**[verified]** Extensions on the mint: `metadataPointer`, `transferFeeConfig`, `tokenMetadata`. There is no permanent-delegate, pausable, transfer-hook, or non-transferable extension on SI.

**[verified]** `transferFeeBasisPoints` is 100 on both the older and newer fee objects (epoch 1039), so the fee is 1%. `maximumFee` is `1000000000000000` raw (1,000,000,000 SI), the same size as the original mint, so the cap does not bind before the whole supply. At context slot 456191784, `withheldAmount` on the mint is `2082526239` raw (2,082.526239 SI).

**[verified]** Current `withdrawWithheldAuthority` and `transferFeeConfigAuthority` are both `5KXDF6QnqhBj72hDtJNkkpFaQVUfbFXNybMsp3DiK6tD`. In the creation transaction's `initializeTransferFeeConfig`, the withdraw authority is already that address, and the config authority is `WLHv2UAZm6z4KyaaELi5pjdbJh6RESMva1Rnn8pJVVh`. The config authority on the mint today is different. The transaction that changed it was not located.

**[heuristic]** One wallet can withdraw the withheld fees, and the current config authority can change the rate. That is a key-person signal on the tax. It is separate from mint and freeze, which are cleared.

**[verified]** `metadataPointer.authority` is null and `metadataAddress` is the mint itself. `tokenMetadata.updateAuthority` is `WLHv2UAZm6z4KyaaELi5pjdbJh6RESMva1Rnn8pJVVh`.

### Quote asset (NVDAx)

The CPMM quote mint is `Xsc9qvGR1efVDFGLrVsmkzv3qi45LTBjeUKSPmx9qEh` (NVIDIA xStock, 8 decimals). These keys are on NVDAx, not on SI.

**[verified]** NVDAx has `permanentDelegate.delegate` = `5aMNNLQJwAEeoemTEMkv5NVjqKwvvefRYCQ5Z67HFvEq`, a freeze authority (`JDq14BWvqCRFNu1krb12bcRpbGtJZ1FLEakMw6FdxJNs`), a mint authority, and `pausableConfig` with `paused: false`. `transferHook.programId` is null. `scaledUiAmountConfig.newMultiplier` is `1.001701196801074`, effective timestamp `1789000200` (10 Sep 2026, 00:30:00 UTC), which is before this run, so the scaled amount is what explorers show.

**[verified]** The creation log says `Mint has a permanent delegate, so tokens in this account may be seized at any time` while the inner instruction is initializing the NVDAx vault. The warning attaches to NVDAx.

## Metadata

**[verified]** On-mint name `Super Inu`, symbol `SI`, URI `https://m.rapidlaunch.io/m/if5VgbxOC`.

**[verified]** GET of that URI returned `name` Super Inu, `symbol` SI, `showName` true, `image` host `axiomtrading.sfo3.cdn.digitaloceanspaces.com`, and `twitter` `https://truthsocial.com/@realdonaldtrump/117309799376551907`. The JSON is data from that host.

**[heuristic]** Update authority is set, so the name and URI can be changed by `WLHv2…`. The metadata pointer authority is null, so the pointer is not movable.

**[heuristic]** Launch-tool fingerprint: the URI host is `m.rapidlaunch.io`, the creation transaction sends 0.07 SOL to `rapidXMVLw5uBieKHDGvF9k4xSSDXyD2FC5wLTAajaJ`, and an unlabeled program in that transaction is `rapidV5DAKtecYzetwCNoxXWHpezjeg4hGt9BGiVcGW`. The shared `rapid` prefix is the fingerprint.

## Creation transaction

**[verified]** Signature `5X4SGFFUFZCrxTeabiQFJGKGp1QjkEhH25w1XrXJryJ2GASWctDipDnFX7eHidMhHyjxWJc3x6TpvPJLrATdDSe6`, slot `449099389`, `blockTime` `1790005495` (21 Sep 2026, 15:44:55 UTC, 17:44:55 CEST). `meta.err` is null. `meta.fee` is 10,000 lamports. Two signatures: the fee payer and the mint account. Version is 1. `transactionConfig.computeUnitLimit` is 1,400,000 and `priorityFee` is null. Priority fee derived as `10000 - 5000 * 2 = 0`.

**[verified]** Fee payer `9pkJqJLtefuna3GLtKZyF5DBWwEGUdxZUSGCmf5jZhA9`.

**[verified]** `getSignaturesForAddress` on LaunchLab pool `5aBH535rESkhjVb2nEaMPfyjiTw3uK57vE6c5uhZHjPY` returned 610 signatures; the oldest is this same creation signature. The pool PDA matches seeds `["pool", SI mint, NVDAx mint]` under Raydium LaunchLab.

Outer instructions:

1. **System Program** `transfer`. 70,000,000 lamports (0.07 SOL) from the fee payer to `rapidXMVLw5uBieKHDGvF9k4xSSDXyD2FC5wLTAajaJ`.
2. **Raydium LaunchLab** (`LanMV9sAd7wArD4vJFi2qDdfnVhFxYSUg6eADduJ3uj`). Log: `Instruction: InitializeWithToken2022`. Inner instructions create the pool and the NVDAx vault, initialize the metadata pointer, initialize the transfer fee, `initializeMint2` (decimals 6, mint authority `WLHv2…`), `mintTo` of 1,000,000,000 SI into curve vault `quqm157mHbG3KNibBAefv9FuFB27EP7onBrL26XHNfd`, and `setAuthority` mintTokens to null. Platform config account in the instruction: `6BwHHDg3u1854jC8PDLXvR4spTcLNaoBxLJNGC4nTESt` (label: StonkFun LaunchLab platform config).
3. **Unlabeled program** `rapidV5DAKtecYzetwCNoxXWHpezjeg4hGt9BGiVcGW`. Inner `transferChecked` of NVDAx from the fee payer to three owners (below).
4. **Associated Token Account** `createIdempotent` for the fee payer's SI account `39d4ZS4WRLuaJzMPD9meCd98ExA8PSHyDEqYcarXTjdv`.
5. **Unlabeled program** `rapidV5…` again. Inner instructions include a LaunchLab buy: NVDAx into the curve, SI out to the fee payer, and an NVDAx transfer to a fee account.

**[verified]** NVDAx moved inside this transaction (8 decimals):

| To owner | Amount |
|---|---:|
| `tQi75x9GeqsDeFdPVdwn6fwfiNLog53STe56Wjt4MVj` | 1.117886 |
| `8pNSgAe9JPi7fSodDSAXmjY9tbkPZQ5TpdR6LddhJtse` | 0.941413 |
| `7K1ygpyD3EXtyfnQ1X8Vsk5tn2PR7Whah96dCZoR7uZ` | 1.008513 |
| Curve vault (owner `WLHv2…`) | 3.64218805 |
| Token account `HGfJN3HdBs5AmBqoN1Y4eSs4D4hKy5MtkiTe7Rn7wn3S`, taken from the curve vault | 0.03642188 |

**[derived]** 1.117886 + 0.941413 + 1.008513 + 3.64218805 = 6.71000005 NVDAx, which matches the fee payer's NVDAx balance dropping from 6.71000005 to 0. The 0.03642188 sent to `HGfJN3…` is about 0.990% of (3.64218805 + 0.03642188).

**[verified]** SI `transferChecked` to the fee payer: gross `200281990292724` raw (200,281,990.292724 SI). Post balance on that account: `198279170389796` raw (198,279,170.389796 SI).

**[derived]** The post balance is 99% of the gross transfer. The 1% gap is 2,002,819.902928 SI, the transfer fee on the fee payer's own buy.

Curve vault post balance of SI: 799,718,009.707276. Gross buy plus that vault balance equals 1,000,000,000.

## Same-slot activity

**[verified]** `getBlock` on slot 449099389 with `transactionDetails: "signatures"` returned 1,224 signatures. The creation signature is at 0-based index 1192. The previous signature is `66sWAB49MK57CK1zXN38HJ4nwgm38mPT7xQUMHGtyxyJbgakKDBiD6FP9G9vRveB7N7Qhr4m8zCVSbkT4P7GiH8L`. The next is `5jUWKqzrzE42fEKeRuqv9vdygSVsxEKufAcmcGLvFVKDqNYhYS67RGjFfHhn9KGB4voaiKbkvczd2NAzcthVMeWY`. Slots 449099388 and 449099390 were not fetched. The other 1,221 signatures in the creation slot were not decoded.

### Funding transaction

**[verified]** `66sWAB49…` is in the same slot. Four signers: the fee payer, `tQi75…`, `8pNSg…`, and `7K1yg…`. `meta.fee` is 20,000 lamports (4 × 5,000). `meta.err` is null.

**[verified]** System transfers of 2.15 SOL, 1.81 SOL, and 1.94 SOL from those three wallets to the fee payer, and 0.001 SOL from the fee payer to `AstraT7S7BfzPFVBQRKNRSkv17Rq3pmaWqGyGXzp4NZ5`.

**[verified]** Pre-balances on that receipt: fee payer 244.386310 SOL, `tQi75…` 157.027740469 SOL, `8pNSg…` 99.510575650 SOL, `7K1yg…` 61.282076786 SOL.

**[verified]** An inner Raydium CLMM swap (`CAMMCzo5YL8w4VFF8KVHrK22GGUsp5VTaW7grrKgrWqK`) moves 12.9 SOL of wSOL in and 6.71000005 NVDAx out to the fee payer. The 12.9 SOL equals 2.15 + 1.81 + 1.94 + 7.00.

### Buy transaction

**[verified]** `5jUWKqzr…`, same slot, three signers (the three wallets, not the fee payer). Each sends SOL to `rapidXMV…`: 0.0215, 0.0181, and 0.0194, which are 1% of 2.15, 1.81, and 1.94. `transactionConfig.computeUnitLimit` is 1,400,000, `priorityFee` null, `meta.fee` 15,000 lamports.

**[verified]** Gross SI bought (transferChecked) and post-fee balances:

| Wallet | Gross SI | Post-fee balance | Share of 1,000,000,000 |
|---|---:|---:|---:|
| `tQi75…` | 47,289,003.501516 | 46,816,113.466500 | 4.682% |
| `8pNSg…` | 36,022,269.381084 | 35,662,046.687273 | 3.566% |
| `7K1yg…` | 35,257,477.337553 | 34,904,902.564177 | 3.490% |
| Fee payer (from the creation receipt, unchanged by this one) | 200,281,990.292724 | 198,279,170.389796 | 19.828% |

**[derived]** Post-fee sum of the four wallets: 315,662,233.107746 SI, which is 31.566% of the 1,000,000,000 minted. The three buyers alone hold 117,383,062.717950 SI (11.738%).

### The next 76 seconds

**[verified]** First SI balance decrease for `tQi75…` is slot 449099552, `blockTime` 15:45:39 UTC (17:45:39 CEST), 44 seconds after creation.

**[verified]** Successful receipts in which the SI balance falls: 36 for `tQi75…`, 12 for `8pNSg…`, 11 for `7K1yg…`.

**[verified]** SI post-balance reaches 0 at:

| Wallet | Time (UTC) | Seconds after creation | SOL post-balance on that receipt |
|---|---|---:|---:|
| `tQi75…` | 15:45:57 | 62 | 187.494747169 |
| `8pNSg…` | 15:46:05 | 70 | 103.430182995 |
| `7K1yg…` | 15:46:11 | 76 | 67.063538284 |

**[derived]** SOL change versus the pre-balance on the funding transaction: `tQi75…` +30.467007 SOL, `8pNSg…` +3.919617 SOL, `7K1yg…` +5.781461 SOL. Combined +40.168086 SOL. Fees are already inside the balances.

**[verified]** In slot 449099606 (15:45:53 UTC), receipt `5BH11zCK7hiScUaXkeCx2UaFxw6JPecNMAibCKVoKQ77JnP35w4kQY9AdgJu7t8RSBwKzWAZbGHczGKab6zfdEBP` moves `tQi75…` from 0 to 1,591,716.724234 SI. The wallet's SOL account is not in that receipt. Other receipts in that same second show `8pNSg…` and `7K1yg…` SI balances falling while `tQi75…` rises, before `tQi75…` sells again down to 0 at 15:45:57. Those are transfers among the three, then a sale.

**[heuristic]** The 120-second window shows a one-way exit of the three wallets' SI, including a pass of some balance onto `tQi75…` before the last sales. It does not show them buying SI again in that window. Later signatures exist on these token accounts (hours after launch) and were not decoded, so this says nothing about those later receipts. `8pNSg…` and `7K1yg…` SI accounts no longer exist (`getAccountInfo` returned null); `tQi75…`'s account still has a later history.

## Pool

Snapshot context slots 456191783–456191786 on 12 Oct 2026.

**[verified]** Pool `B4VFURUHHzyt8YzBGBV9jiarBvjh1EbMAbRNBnNqaxUD` is owned by Raydium CPMM. `token_0` is NVDAx, vault `8MPZtAjUjKx5F7jQSMaefy1hjKWP6uw3JtUUcDdYh7ub`. `token_1` is SI, vault `DcpYAKLzwFV8G1Q9V5sTn5m4MLqdaB3yJ4Qm87QgXNd8`. Both vaults are owned by `GpMZbSM2GgvTKHJirzeGfMFoaZ8UR2X7F4v8vHTvxFbL`. `pool_creator` is `5CEbueQnq1Ym2uSSx2xXds3jQAqT1BDnkA59RZobSPAG`. `open_time` is `1790005501` (21 Sep 2026, 15:45:01 UTC, 17:45:01 CEST). Dexscreener and Raydium's public pool API both name this address; the owner and mints above are the chain check.

**[verified]** Amm config `CRRS5ieQmBrZjWhcj99JuGrT5tyuWDaGAXLXLFjbAtjQ`: `trade_fee_rate` 2500, `protocol_fee_rate` 120000, `fund_fee_rate` 40000, `create_pool_fee` 150000000 lamports (0.15 SOL), `creator_fee_rate` 10000 (the u64 immediately after `fund_owner`). The next u64 is 50000 and is left unnamed. `protocol_owner` is `ProCXqRcXJjoUd1RNoo28bSizAA6EEqt9wURZYPDc5u`. `fund_owner` is `FUNDduJTA7XcckKHKfAoEnnhuSud2JUCUZv6opWEjrBU`.

**[derived]** Total CPMM fee rate = (2500 + 10000) / 1,000,000 = 1.25%. Protocol and fund rates, read as shares of the trade fee, are 12% and 4% of the 0.25% trade fee. Raydium's API `feeRate` on this pool was 0.0125 and `config.creatorFeeRate` was 10000, which matches those integers.

**[verified]** Raw vault balances at this slot: NVDAx `145954451710` (1,459.54451710), SI `15533584605314` (15,533,584.605314). Protocol fees on the pool: 0 and 0. Fund fees: NVDAx `1910132` (0.01910132), SI `249193959` (249.193959). An earlier read in the same session had non-zero protocol fees; they were 0 at this slot.

**[derived]** Reserves = vault − protocol fees − fund fees. NVDAx reserve 1,459.52541578. SI reserve 15,533,335.411355. Price = 1,459.52541578 / 15,533,335.411355 = 0.00009396085 NVDAx (raw) per SI.

**[derived]** NVDAx has a scaled-UI multiplier of 1.001701196801074, so the displayed quote reserve is about 1,462.008 and the displayed price is about 0.000094121 NVDAx per SI. The constant-product math uses the raw reserves, not `uiAmountString`.

**[derived]** Exact-in swap of 1 raw NVDAx (100,000,000 raw) at these reserves: fee = 1,250,000 raw (exactly 1.25%), amount in after fee = 98,750,000, SI out = 10,502.590256. With a zero fee the same input yields 10,635.443412 SI. The fee reduces the output by 132.853 SI on this size.

**[heuristic]** SI's own 1% transfer fee is a second charge, taken when SI moves, and is not part of the 1.25% CPMM rate. This run did not fold the two into one number.

## LP

**[verified]** LP mint `Dc2nrzZ5UGNXj1youxxuB9Bjf6jSzy8C7zJUkZtpvLdq` (SPL Token, 9 decimals). Mint `supply` `955147629198`. Pool `lp_supply` `955147629298`. The difference is 100 raw units.

**[derived]** Burned-by-supply-gap is dust (100 raw units on ~9.55×10^11). Destroying LP is not what accounts for the large holder below.

**[verified]** Largest LP token account `2sRPWbzFRRfe8XNGDgfJJCU4WmmLD3gVVpLgho7fRc2H` holds 953.701859956 LP. Owner `3f7GcQFG397GAaEnv51zR6tsTVihYRydnydDD1cXekxH`. That owner account is owned by the System Program, space 0. Next resolved holders: 1.151620326 LP owned by `GCrtMQU5UvUMWVbRALxdN9TAUN3vsC3BRLkLLd5Uh3px`, and 0.13473479 LP owned by `819wN3vcVfdku2YL29vY5RjxR6eEbk7zQCqLRj4rgmbU`. Two further accounts hold 0.111047027 and 0.048367099; their owners were not resolved.

**[derived]** 953.701859956 / 955.147629198 = 99.849% of LP mint supply sits in the system account. The other resolved holders are about 0.135% combined.

**[heuristic]** Raydium's public API `burnPercent` for this pool was 99.84, in line with that share. This run did not decode a burn instruction or a lock program. A system account can sign later. The recent signatures sampled on the LP token account invoked `GRu1LbYk4taq1KstBnB7qgxdDPo5g2JrKxuB3VxdsjhG` with logs `Instruction: SampleMint` and `SampleMintExt`, and the LP balance did not change in that receipt. The graduation transaction that created the pool was not decoded (candidates in slots 449099432 and 449099496 were not a decoded CPMM initialize).

**[verified]** Creator-fee field: `pool_creator` `5CEbue…` and `creator_fee_rate` 10000. No creator-fee payout receipt was opened in this run.

## Holders

`getTokenLargestAccounts` returns 20 accounts. Supply used: 996,024,042.883960 SI. The pool vault in this list held 15,533,665.656602 SI (it ticked slightly by the later pool read). Excluded from the holder total: that CPMM vault.

**[verified]** Largest non-pool account holds 40,710,068.452866 SI, owner `mXvJVLBBVyi9o7E2rUAdnNzUyA7gFtabEjdKfCWRTtj`.

**[derived]** That wallet is 4.09% of supply. The 10 largest non-pool holders in the list sum to 18.14% of supply, under the 30% line. The 19 non-pool accounts in the top 20 sum to 26.10%. The creation fee payer and the three creation-slot wallets are not among these 20 owners.

**[heuristic]** First-SOL-inflow clustering was not run (page budget not started). Not evaluated.

Top non-pool owners in the list:

| SI | Owner |
|---:|---|
| 40,710,068.45 | `mXvJVLBBVyi9o7E2rUAdnNzUyA7gFtabEjdKfCWRTtj` |
| 22,166,288.62 | `5cNbJRngzJR1omd1M7G2YpgZsQ8Z5VunhwHDamrpMWvd` |
| 20,658,659.83 | `2heJbC32Tpfcb3nbUb5ER61K11FGZVfVGtVnDm6LDogF` |
| 18,775,673.78 | `9w5Zu53ohJWD2rGCLMwaexZ6xHx1FcezuZDAmLkfnNir` |
| 16,600,190.77 | `FpyJSVbf1FkFEV5AamCFuANZHJm1TpcbbPV2UXT8YGEv` |

## Fee path

Sampled from the withdraw authority's SI account `7eK3C87xphbHRMg72CCg3j7A7LB2Ciivhakc3QHxkn9w` (owner `5KXDF6…`). At context slot 456190720 that account held 79,278.267189 SI and `withheldAmount` 0 on the account itself.

**[verified]** `2jpb76zCvKD9jAyiRsFRAJPN3zhrRCCBczBy47dtoJdNut26toonJn29uzxknUdHwtKTAQQZ1YFKZ5CFqT8R7c4T` at 12 Oct 2026 21:12:07 UTC (23:12:07 CEST). Signer `5KXDF6…`. Log: `TransferFeeInstruction: WithdrawWithheldTokensFromMint`. The authority's SI balance rose by 2,708.169940. `meta.fee` 5,000 lamports.

**[verified]** `2X8nzBnALQBbcRn9qLLmo5LhsUemJZLYVngWgxwAR8SMJyVuiPGjSKmVAHy4A52CapZ93Fv1WV4KKxDyDNgkgs7z` at 21:12:08 UTC, same signer. Log: `Instruction: SwapBaseInput` on Raydium CPMM. Authority SI balance fell by 2,708.169940. The pool vault's SI balance rose by 2,681.088240. The authority received 0.248815 NVDAx.

**[derived]** 2,708.169940 − 2,681.088240 = 27.081700 SI, which is 1.000% of 2,708.169940. The 1% transfer fee applied to the authority's own sell. The NVDAx stayed with `5KXDF6…` in this receipt (account `2ARST2YrqnLAX9Dy28db1D2zFp1SYf6ofTfDaH8uAa4c`).

**[verified]** The next receipts on that NVDAx account in the following minutes are further sells of other mints by the same signer (LaunchLab `SellExactIn`), not a transfer that splits the 0.248815 NVDAx. A payout split was not in this sample of signatures. No ratio is reported.

## Trading patterns

**[verified]** Of the decoded receipts in the exit window, 42 share version-1 `computeUnitLimit` 800,000 and `priorityFee` 100,000 lamports (`meta.fee` 105,000, one signature, so the derived priority fee is 100,000 lamports). The creation and the three-wallet buy use limit 1,400,000 and a null priority fee.

**[heuristic]** Identical limit and priority fee across 42 receipts is a shared-settings signal. The four signatures on the funding transaction are the stronger signal.

**[verified]** The funding, creation, and same-slot buy receipts contain no system transfer to the listed Jito tip accounts. The 0.001 SOL to `AstraT7S…` is an unlabeled transfer.

**[heuristic]** In the first 76 seconds the three wallets' SI goes to zero, with a pass onto `tQi75…` in between, and no second buy from them in that window. That is a fast-exit signal in the sample. A buy-then-sell loop against the pool, in the wash sense, was not what these receipts show. Slots `S+1` and `S+2` were not scanned for other buyers.

## Deployer

**[verified]** Creation fee payer `9pkJqJLtefuna3GLtKZyF5DBWwEGUdxZUSGCmf5jZhA9`. Pre-balance on the funding receipt: 244.386310 SOL.

**[verified]** `getSignaturesForAddress` on that wallet with `before` set to the creation signature returned the funding signature first, then 14 older signatures (the call asked for 15). Those 14 were not decoded. Other launches were not evaluated.

## Not evaluated

- Account-key scan of the rest of slot 449099389, and of slots 449099388 and 449099390.
- Whether these wallets or these SOL amounts (2.15 / 1.81 / 1.94) appear on other mints. The notes on this launch describe 11 nearby launches; this run did not check that.
- First SOL inflow for the current top holders.
- The 14 deployer signatures before the funding transaction.
- A quote-token split after the fee wallet's swap, including any 97.5 / 2.5 ratio. The sample stops at the swap.
- The CPMM initialize transaction and a lock-program proof for the LP system account.
- USD value of the reserves.

## What matched the known SI notes

| Note | This run |
|---|---|
| Token-2022, 1% transfer fee | Reproduced (basis points 100). |
| Withdraw authority `5KXDF6…` | Reproduced, on the mint and in `initializeTransferFeeConfig`. |
| Mint and freeze authorities off | Reproduced (`null` now, and `setAuthority` mintTokens to null in the creation receipt). |
| Wallets `tQi75…`, `8pNSg…`, `7K1yg…`, about 31.6% at block 0 | Reproduced: post-fee share with the fee payer is 31.566%. |
| Graduation pool `B4VFURUH…` | Reproduced. CPMM owner and both mints checked. |
| Funding amounts 2.15 / 1.81 / 1.94 SOL, four signers, then the three-wallet buy, same block | Reproduced. Signatures `66sWAB49…` and `5jUWKqzr…`. |
| Sells from +44s to +76s, 36 / 12 / 11 decreasing receipts | Reproduced. |
| Net SOL about +30.47, +3.84, +5.78, together about +40 | +30.467, +3.920, +5.781, together +40.168. The 8pNSg figure is 0.08 SOL higher than the note, using the post-balance on the receipt where that wallet's SI hit 0. |
| Same pattern on 11 other launches | Not evaluated. |
