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

## Create

Create a transaction:
```bash
nunchuk tx create --wallet <wallet-id> --to <address> --amount <satoshis>
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
- If no key option is provided, `tx sign` auto-detects matching stored keys for the wallet and signs with all of them.
- Use `tx get --json` to retrieve the current pending PSBT before signing externally.
- Use `--json` when the user needs exact machine-readable output.

## Gotchas

- `tx sign` skips keys that already signed the PSBT.
- `tx sign --psbt` cannot be used with `--xprv` or `--fingerprint`.
- `tx broadcast` requires a fully signed transaction, status is `READY_TO_BROADCAST`.
