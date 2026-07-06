---
name: nunchuk-wallet-transactions
description: Create, preview, sign, inspect, list, and broadcast Bitcoin transactions, including fee control (manual fee rate, fee levels, subtract fee, send all, anti-fee sniping) and coin selection (spend specific coins, spend from a tag or collection, change-coin tags). Use when the user wants to send funds, preview or adjust fees, choose which coins to spend, sign a transaction, broadcast it, or inspect wallet transaction history.
---

# Nunchuk Wallet Transactions

If auth or network setup is the blocker, use `nunchuk-setup`.

If the user wants to lock coins or manage coin tags and collections, use `nunchuk-coin-control`.

## Default workflow

Progress:
- [ ] Preview with `tx draft` (optional, when the user wants to check fee or inputs first)
- [ ] Create the transaction
- [ ] Sign the transaction
- [ ] Broadcast the transaction

If the user explicitly asks for only preview, only sign, only broadcast, or only inspect, do only that step.

## Preview

`tx draft` builds the same transaction `tx create` would (same coin selection, same fee) but never creates, uploads, or stores anything. It accepts the same options as `tx create`, plus `--fiat <code>`:

```bash
nunchuk tx draft --wallet <wallet-id> --to <address> --amount 100000
nunchuk tx draft --wallet <wallet-id> --to <address> --send-all
nunchuk tx draft --wallet <wallet-id> --to <address> --amount 100000 --fiat USD
```

The draft shows the fee rate, estimated fee, total amount, change output, and the input coins that would be spent.

## Create

Create a transaction:
```bash
nunchuk tx create --wallet <wallet-id> --to <address> --amount <satoshis>
```

Send the entire wallet balance (no change; the recipient receives balance minus fee):
```bash
nunchuk tx create --wallet <wallet-id> --to <address> --send-all
```

Make the recipient pay the fee (receives `amount - fee`):
```bash
nunchuk tx create --wallet <wallet-id> --to <address> --amount 100000 --subtract-fee
```

Pin `nLockTime` to the current block height (anti-fee sniping):
```bash
nunchuk tx create --wallet <wallet-id> --to <address> --amount 100000 --anti-fee-sniping
```

## Fees

`tx create` auto-estimates the fee rate. Override it per transaction:

```bash
# Manual fee rate in sat/vB (fractional accepted)
nunchuk tx create --wallet <wallet-id> --to <address> --amount 100000 --fee-rate 1.5

# Pick the estimate level for this transaction
nunchuk tx create --wallet <wallet-id> --to <address> --amount 100000 --fee-level priority
```

Show the current recommended rates for the three levels (`priority`, `standard`, `economy`):
```bash
nunchuk tx fees
```

Save a default fee level for the account (used when neither `--fee-rate` nor `--fee-level` is given):
```bash
nunchuk config fee-rate set priority
nunchuk config fee-rate get
nunchuk config fee-rate reset
```

Precedence: `--fee-rate` > `--fee-level` > saved account default > built-in `economy`.

## Choose Coins

Spend exactly specific coins (manual selection, no automatic top-up):
```bash
nunchuk tx create --wallet <wallet-id> --to <address> --amount 100000 --coin <txid>:<vout> --coin <txid>:<vout>
```

Sweep only specific coins:
```bash
nunchuk tx create --wallet <wallet-id> --to <address> --send-all --coin <txid>:<vout>
```

Restrict automatic selection to coins carrying a tag, coins in a collection, or both (intersection):
```bash
nunchuk tx create --wallet <wallet-id> --to <address> --amount 100000 --from-tag kyc
nunchuk tx create --wallet <wallet-id> --to <address> --amount 100000 --from-collection "Exchange A"
nunchuk tx create --wallet <wallet-id> --to <address> --amount 100000 --from-tag kyc --from-collection "Exchange A"
```

Control which tags the change coin inherits (default: all tags of the input coins):
```bash
nunchuk tx create --wallet <wallet-id> --to <address> --amount 100000 --change-tags none
nunchuk tx create --wallet <wallet-id> --to <address> --amount 100000 --change-tags kyc,cold
```

Find coin outpoints, lock coins, and manage tags with `nunchuk-coin-control` (`coin list`, `coin lock`, `coin tag`).

## What Miniscript Signing Path Means

A Miniscript wallet can have multiple valid spending branches.

The active signing path determines:
- which signers are required
- whether a locktime is required
- whether a sequence value is required
- whether one or more hash preimages are required

Use `nunchuk miniscript inspect` to view the possible policy paths before spending, or when reviewing a template before wallet creation:
```bash
nunchuk miniscript inspect --wallet <wallet-id>
nunchuk miniscript inspect --descriptor "wsh(...)"
nunchuk miniscript inspect --miniscript "multi(2,key_0_0,key_1_0,key_2_0)"
nunchuk miniscript inspect --miniscript "multi_a(2,key_0_0,key_1_0,key_2_0)" --address-type TAPROOT
nunchuk miniscript inspect --miniscript "or_d(multi(2,key_0_0,key_1_0,key_2_0),and_v(v:pk(key_3_0),after(1735689600)))"
```

Use `--locktime` and `--sequence` when you need to know which path is currently satisfiable:
```bash
nunchuk miniscript inspect --wallet <wallet-id> --locktime 1735689600 --sequence 144
```

Use `nunchuk tx get` to inspect the live transaction-selected path:
```bash
nunchuk tx get --wallet <wallet-id> --tx-id <tx-id>
```

`tx get` can show:
- `Required signers`
- `Locktime`
- `Sequence`
- `Hash preimages`
- `Path signers`

`READY_TO_BROADCAST` depends on the selected Miniscript path being fully satisfied, not on wallet `m-of-n` alone.

For Miniscript, choose a specific path if needed:
```bash
nunchuk tx create --wallet <wallet-id> --to <address> --amount 100000 --miniscript-path 0
```

For Taproot wallets, key-path spend is the default when available. Force script-path spend when needed:
```bash
nunchuk tx create --wallet <wallet-id> --to <address> --amount 100000 --taproot-script-path
nunchuk tx create --wallet <wallet-id> --to <address> --amount 100000 --taproot-script-path --miniscript-path 0
```

For Miniscript, attach required preimages when the selected path needs them:
```bash
nunchuk tx create --wallet <wallet-id> --to <address> --amount 100000 --preimage <32-byte-hex>
```

## Sign

Sign with all matching stored keys:
```bash
nunchuk tx sign --wallet <wallet-id> --tx-id <tx-id>
```

Sign with a specific stored key:
```bash
nunchuk tx sign --wallet <wallet-id> --tx-id <tx-id> --fingerprint <xfp>
```

Attach Miniscript preimages while signing:
```bash
nunchuk tx sign --wallet <wallet-id> --tx-id <tx-id> --preimage <32-byte-hex>
```

Merge an externally signed PSBT:
```bash
nunchuk tx sign --wallet <wallet-id> --tx-id <tx-id> --psbt <signed-psbt-base64>
```

## Inspect

List transactions:
```bash
nunchuk tx list --wallet <wallet-id>
```

Get one transaction:
```bash
nunchuk tx get --wallet <wallet-id> --tx-id <tx-id>
```

## Broadcast

Broadcast a fully signed transaction:
```bash
nunchuk tx broadcast --wallet <wallet-id> --tx-id <tx-id>
```

## Common requests

User asks: send `100 USD` to an address:
```bash
nunchuk tx create --wallet <wallet-id> --to <address> --amount 100 --currency USD
```

User asks: send `250 USD` to an address, then sign and broadcast:
```bash
nunchuk tx create --wallet <wallet-id> --to <address> --amount 250 --currency USD
nunchuk tx sign --wallet <wallet-id> --tx-id <tx-id>
nunchuk tx broadcast --wallet <wallet-id> --tx-id <tx-id>
```

User asks: send everything in the wallet:
```bash
nunchuk tx create --wallet <wallet-id> --to <address> --send-all
```

User asks: check the fee before sending:
```bash
# Preview, then create with the same locked-in rate.
nunchuk tx draft --wallet <wallet-id> --to <address> --amount 100000
nunchuk tx create --wallet <wallet-id> --to <address> --amount 100000 --fee-rate <previewed-rate>
```

User asks: send with the fastest confirmation:
```bash
nunchuk tx create --wallet <wallet-id> --to <address> --amount 100000 --fee-level priority
```

User asks: spend only KYC coins:
```bash
nunchuk tx create --wallet <wallet-id> --to <address> --amount 100000 --from-tag kyc
```

User asks: spend a specific coin:
```bash
# Get outpoints from coin list first (see nunchuk-coin-control).
nunchuk coin list --wallet <wallet-id>
nunchuk tx create --wallet <wallet-id> --to <address> --amount 100000 --coin <txid>:<vout>
```

User asks: sign with a hardware wallet or external signer:
```bash
# Get the current pending PSBT.
nunchuk tx get --wallet <wallet-id> --tx-id <tx-id> --json

# Sign that PSBT externally, then merge it back.
nunchuk tx sign --wallet <wallet-id> --tx-id <tx-id> --psbt <signed-psbt-base64>
```

## Defaults

- `tx create` treats `--amount` as satoshis by default.
- Use `--currency` for fiat or BTC amounts.
- `tx create` estimates the fee rate automatically; precedence when overriding is `--fee-rate` > `--fee-level` > saved account default > `economy`.
- Automatic coin selection skips locked coins; when the transaction produces change, the change coin inherits the input coins' tags unless `--change-tags` overrides it.
- `tx draft` never creates, uploads, or stores anything — safe to run any time.
- For Taproot wallets, `tx create` uses key-path signing by default when available.
- If no key option is provided, `tx sign` auto-detects matching stored keys for the wallet and signs with all of them.
- Use `tx get --json` to retrieve the current pending PSBT before signing externally.
- Use `--json` when the user needs exact machine-readable output.

## Gotchas

- `--coin` is repeatable and spends **exactly** the listed coins — no subset optimization, no automatic top-up; a shortfall fails with insufficient funds.
- `--coin` spends a coin even if it is locked; locks only guard automatic selection.
- `--coin` cannot be combined with `--from-tag` or `--from-collection`.
- `--from-tag` and `--from-collection` given together intersect: only coins matching both qualify.
- `--send-all` implies `--subtract-fee` and ignores `--amount` (with a warning).
- `--subtract-fee` is rejected when the amount cannot cover the fee or the reduced output would be dust.
- `--change-tags` must be `none` or a subset of the tags on the input coins — it copies existing classification, it never invents tags.
- A draft's auto-estimated fee rate can change before `tx create` runs — pass `--fee-rate` to lock the previewed rate.
- `tx sign` skips keys that already signed the PSBT.
- `tx sign --psbt` cannot be used with `--xprv` or `--fingerprint`.
- `--preimage <32-byte-hex>` is required when the chosen Miniscript branch includes hash locks.
- `--miniscript-path <index>` is for choosing a specific branch when the user does not want the default satisfiable path selection.
- Use `--taproot-script-path` when the user wants to spend a Taproot wallet through a script path instead of the default key path.
- `tx broadcast` requires a fully signed transaction, status is `READY_TO_BROADCAST`.
