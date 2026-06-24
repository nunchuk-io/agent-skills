---
name: nunchuk-wallet-transactions
description: Create, sign, inspect, list, and broadcast Bitcoin transactions. Use when the user wants to send funds, sign a transaction, broadcast it, or inspect wallet transaction history.
---

# Nunchuk Wallet Transactions

If auth or network setup is the blocker, use `nunchuk-setup`.

## Default workflow

Progress:
- [ ] Create the transaction
- [ ] Sign the transaction
- [ ] Broadcast the transaction

If the user explicitly asks for only sign, only broadcast, or only inspect, do only that step.

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

## Create

Create a transaction:
```bash
nunchuk tx create --wallet <wallet-id> --to <address> --amount <satoshis>
```

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
- `tx create` estimates the fee rate automatically.
- For Taproot wallets, `tx create` uses key-path signing by default when available.
- If no key option is provided, `tx sign` auto-detects matching stored keys for the wallet and signs with all of them.
- Use `tx get --json` to retrieve the current pending PSBT before signing externally.
- Use `--json` when the user needs exact machine-readable output.

## Gotchas

- `tx sign` skips keys that already signed the PSBT.
- `tx sign --psbt` cannot be used with `--xprv` or `--fingerprint`.
- `--preimage <32-byte-hex>` is required when the chosen Miniscript branch includes hash locks.
- `--miniscript-path <index>` is for choosing a specific branch when the user does not want the default satisfiable path selection.
- Use `--taproot-script-path` when the user wants to spend a Taproot wallet through a script path instead of the default key path.
- `tx broadcast` requires a fully signed transaction, status is `READY_TO_BROADCAST`.
