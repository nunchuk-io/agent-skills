---
name: nunchuk-wallet-transactions
description: Create, preview, import, sign, inspect, list, and broadcast Bitcoin transactions, including batch payments to several recipients (flags, CSV/JSON file, BIP-21 URIs), fee control (manual fee rate, fee levels, subtract fee, send all, anti-fee sniping), and coin selection (spend specific coins, spend from a tag or collection, change-coin tags). Use when the user wants to send funds to one or many addresses, import a PSBT file created or signed elsewhere, preview or adjust fees, choose which coins to spend, sign a transaction, broadcast it, or inspect wallet transaction history.
---

# Nunchuk Wallet Transactions

If auth or network setup is the blocker, use `nunchuk-setup`.

If the user wants to lock coins or manage coin tags and collections, use `nunchuk-coin-control`.

## Default workflow

Progress:
- [ ] Preview with `tx draft` (optional, when the user wants to check fee or inputs first)
- [ ] Create the transaction (or import a PSBT file the user already has)
- [ ] Sign the transaction
- [ ] Broadcast the transaction

If the user explicitly asks for only preview, only import, only sign, only broadcast, or only inspect, do only that step.

## Preview

`tx draft` builds the same transaction `tx create` would (same coin selection, same fee) but never creates, uploads, or stores anything. It accepts the same options as `tx create`, plus `--fiat <code>`:

```bash
nunchuk tx draft --wallet <wallet-id> --to <address> --amount 100000
nunchuk tx draft --wallet <wallet-id> --to <address> --send-all
nunchuk tx draft --wallet <wallet-id> --to <address> --amount 100000 --fiat USD
nunchuk tx draft --wallet <wallet-id> --recipients-file payouts.csv --fiat USD
```

The draft shows the fee rate, estimated fee, total amount, change output, and the input coins that would be spent. With several recipients it lists each one (and what each receives under `--subtract-fee`).

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

## Pay several recipients (batch)

One transaction can pay many addresses: one fee, one signing round. Give the recipients in exactly one of three forms.

A few recipients on the command line (`--recipient` is repeatable; the amount unit is the row's own, else `--currency`, else sat):
```bash
nunchuk tx create --wallet <wallet-id> --recipient <addr1>:100000 --recipient <addr2>:250000
nunchuk tx create --wallet <wallet-id> --recipient <addr1>:100:USD --recipient <addr2>:0.002:BTC --recipient <addr3>:50000:sat
nunchuk tx create --wallet <wallet-id> --recipient <addr1>:0.001 --recipient <addr2>:0.002 --currency BTC
```

Many recipients from a file (CSV or JSON, detected by content):
```bash
nunchuk tx create --wallet <wallet-id> --recipients-file payouts.csv
nunchuk tx create --wallet <wallet-id> --recipients-file payouts.json --currency USD
```

CSV: `address,amount[,currency]` per line; blank lines and `#` comments ignored; an optional `address,amount` header is skipped; leave the currency empty to use `--currency`:
```
address,amount,currency
bc1q...aaa,100000,
bc1q...bbb,250,USD
bc1q...ccc,0.5,BTC
```

JSON: an array of `{ "address", "amount", "currency"? }` (amount may be a number or a string):
```json
[
  { "address": "bc1q...aaa", "amount": "100000" },
  { "address": "bc1q...bbb", "amount": 250, "currency": "USD" }
]
```

A BIP-21 payment URI (from a QR code or invoice) works wherever an address goes. Its `amount` is in BTC and is the row's amount; quote it on the command line:
```bash
nunchuk tx create --wallet <wallet-id> --recipient "bitcoin:bc1q...aaa?amount=0.001" --recipient bc1q...bbb:250000
```

Recipients pay the fee together with `--subtract-fee` (split equally, remainder on the first recipient):
```bash
nunchuk tx create --wallet <wallet-id> --recipients-file payouts.csv --subtract-fee
```

Everything else (`--fee-rate`, `--fee-level`, `--coin`, `--from-tag`, `--from-collection`, `--change-tags`, `--anti-fee-sniping`, miniscript and taproot options) works the same as for one recipient. `--send-all` is single-recipient only.

Output for a batch prints a `Recipients (n):` block; `--json` adds `recipients: [{ address, amount, receives }]`. Single-recipient output is unchanged.

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

Merge an externally signed PSBT (when you know the txid and the PSBT fits on the command line; otherwise use `tx import`):
```bash
nunchuk tx sign --wallet <wallet-id> --tx-id <tx-id> --psbt <signed-psbt-base64>
```

## Import

Import a PSBT file that was built or signed outside Nunchuk (Sparrow, a hardware wallet's SD card, a script):
```bash
nunchuk tx import --wallet <wallet-id> --file payout.psbt
```

The file may be binary, base64 text, or hex text. The txid is derived from the file and checked against the group server:
- not on the server yet → uploaded as a new pending transaction (`action: created`)
- already pending → merged with the server copy so new signatures reach every device (`action: merged`, or `unchanged` when the file adds nothing; still exit 0)

Before uploading, every input must belong to the wallet, be unspent with the amount the PSBT claims, and use the canonical sighash. A PSBT for another wallet, an already-broadcast transaction, a raw signed transaction, or PSBT v2 is rejected. Devices pick the transaction up automatically; continue with `tx sign` / `tx broadcast` as usual.

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

User asks: pay several people at once (payroll, payouts):
```bash
# A few recipients: unit per row, else --currency, else sat.
nunchuk tx create --wallet <wallet-id> --recipient <addr1>:100:USD --recipient <addr2>:250000:sat
# Many recipients: put them in a CSV (address,amount[,currency]) or JSON file; preview first.
nunchuk tx draft  --wallet <wallet-id> --recipients-file payouts.csv --fiat USD
nunchuk tx create --wallet <wallet-id> --recipients-file payouts.csv
```

User asks: pay a payment request / QR code (`bitcoin:` URI):
```bash
nunchuk tx create --wallet <wallet-id> --recipient "bitcoin:<address>?amount=0.001"
```

User asks: import a PSBT signed on a hardware wallet or in Sparrow:
```bash
nunchuk tx import --wallet <wallet-id> --file signed.psbt
nunchuk tx broadcast --wallet <wallet-id> --tx-id <tx-id>   # once READY_TO_BROADCAST
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

# Or, if the signer wrote a file (or the PSBT is large):
nunchuk tx import --wallet <wallet-id> --file signed.psbt
```

## Defaults

- `tx create` treats `--amount` as satoshis by default.
- Use `--currency` for fiat or BTC amounts; it is the default unit for every amount in the invocation, and a `--recipient` row or file row may carry its own unit (`addr:amount:USD`). A BIP-21 URI amount is always BTC.
- Recipients in a batch are paid in the order given; the change output is placed at a random position.
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
- Use exactly one of `--to`, `--recipient`, `--recipients-file`. `--amount` and `--send-all` go with `--to` only; `--send-all` is single-recipient.
- With several recipients `--subtract-fee` splits the fee equally and the first listed recipient pays the remainder; each reduced output is re-checked against dust.
- Duplicate addresses in a batch are rejected (compared by script, so a bech32 address in another letter case still collides); merge the amounts into one row. Errors name the flag (`--recipient #2`) or file row (`payouts.csv line 4`).
- Prefer `--recipients-file` beyond ~10 recipients; the command line has OS length limits. A batch over the 400,000 WU standard limit is rejected at create time; split the file.
- A `bitcoin:` URI must be quoted (`?` and `&` are shell characters); a row may not give both a URI amount and its own amount; `req-` parameters are rejected.
- Batch JSON has `recipients[]` with `amount` and `receives`; `recipientAmount` / `recipient` appear only for a single recipient. In `--json` mode failures are `{ error, message }` objects (`INVALID_PARAM`, `FILE_NOT_FOUND`, `TX_BUILD_FAILED`).
- `tx import` derives the txid from the file and works whether or not the transaction already exists on the server; use it instead of `tx sign --psbt` when you do not know the txid or the PSBT is too large for the command line.
- `tx import` rejects a PSBT whose inputs do not belong to the wallet, whose inputs are already spent, whose UTXO data disagrees with the chain, that uses a non-canonical sighash flag, a raw signed transaction (not a PSBT), and PSBT v2.
- `tx sign` skips keys that already signed the PSBT.
- `tx sign --psbt` cannot be used with `--xprv` or `--fingerprint`.
- `--preimage <32-byte-hex>` is required when the chosen Miniscript branch includes hash locks.
- `--miniscript-path <index>` is for choosing a specific branch when the user does not want the default satisfiable path selection.
- Use `--taproot-script-path` when the user wants to spend a Taproot wallet through a script path instead of the default key path.
- `tx broadcast` requires a fully signed transaction, status is `READY_TO_BROADCAST`.
- Batch recipients (`--recipient`, `--recipients-file`), `bitcoin:` URIs, and `tx import` require `nunchuk-cli >= 0.2.2`.
