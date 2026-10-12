---
name: trading-patterns
description: >
  Read-only signals for wash-style round trips, snipers in the first
  slots after creation, priority fees, Jito tip accounts, and identical
  compute-budget settings. Used by the check skill. Signals, not accusations.
---

# Trading patterns

Look at the receipts you already have from bundle-detection (creation slot and the early sells) and, if that set is thin, the first 30 successful signatures on the LaunchLab pool or the CPMM pool after `open_time`.

## What to measure

**Round trip.** A wallet whose balance of this mint rises in the window and later falls. That is a round trip. If the fall is a `transferChecked` to another wallet in the same cluster, say "transferred", and if the fall is a swap (pool balance of this mint rises), say "sold to the pool". A round trip is a signal. It is not, by itself, wash trading.

**Funding loop.** In the same window, wallet A sends SOL to B and a later receipt sends SOL from B to A. Quote both signatures. One direction only is not a loop.

**Snipers.** Wallets that gain this mint in slot `S` or slots `S+1` and `S+2`, other than the creation fee payer and the co-signers already listed. You only know this if the slot scan in bundle-detection fetched those receipts. If it did not, write "not evaluated" instead of "no snipers".

**Priority fee.** For each early transaction:

- `meta.fee` (lamports)
- signature count
- **derived** priority lamports = `meta.fee - 5000 * signature_count` when that is ≥ 0
- version-1 `transactionConfig.priorityFee` and `computeUnitLimit` when present
- Compute Budget `setComputeUnitPrice` (micro-lamports per compute unit) and `setComputeUnitLimit` when the instruction is parsed

Do not add micro-lamports to lamports. Report the units.

**Jito tip fingerprint.** A system transfer to one of these accounts is a tip payment in that receipt. The list is a public fingerprint for detection:

- `96gYZGLnJYVFmbjzopPSU6QiEV5fGqZNyN9nmNhvrZU5`
- `HFqU5x63VTqvQss8hp11i4wVV8bD44PvwucfZ2bU7gRe`
- `Cw8CFyM9FkoMi7K7Crf6HNQqf4uEMzpKw6QNghXLvLkY`
- `ADaUMid9yfUytqMBgopwjb2DTLSokTSzL1zt6iGPaS49`
- `DfXygSm4jCyNCybVYYK6DwvWqjKee8pbDmJGcLWNDXjh`
- `ADuUkR4vqLUMWXxW9gh6D6L8pMSawimctcNZ5pGwDcEt`
- `DttWaMuVvTiduZRnguLF7jNxTgiMBZ1hyAumKUiL2KRL`
- `3AVi9Tg9Uo68tJfuvoKvqKNWKkC5wPdSSdeBnizKZ6jT`

Any other SOL transfer to a third party stays "unlabeled SOL transfer", with the amount. Do not call it a Jito tip.

**Shared settings.** Group the early receipts by (`computeUnitLimit`, `priorityFee` or the derived priority lamports). **Heuristic** when a group has at least 5 receipts and at least half of the receipts you grouped: "shared compute limit and priority fee across `<n>` receipts". Identical settings are a signal that the transactions were built the same way. The co-signed funding transaction, if you have one, is the stronger signal; say that.

## Thresholds

| Pattern | Report |
|---|---|
| Round trip inside 120 seconds of creation, sold to the pool, no later buy in the window | Fast exit signal for that wallet. |
| Rise and fall that net to roughly zero while both legs hit the pool | Possible circular volume. Call it a signal and show the two signatures. |
| SOL A→B and B→A | Funding-loop signal. |
| Shared compute settings, ≥ 5 receipts | Shared-settings signal. |
| Jito tip account in the receipt | Tip fingerprint, with the amount. |

## Report lines

```text
**[verified]** <n> receipts in the first two minutes share computeUnitLimit <limit> and priorityFee <lamports> lamports.
**[verified]** No system transfer in the funding, creation, or same-slot buy receipts goes to a listed Jito tip account. A separate transfer of <sol> SOL to `<addr>` is unlabeled.
**[heuristic]** The three wallets' mint balance goes to zero by +<sec>s and this window has no second buy from them. That is a one-way exit in the sample, which is a different pattern from a buy-sell loop.
```
