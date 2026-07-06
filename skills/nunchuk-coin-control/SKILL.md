---
name: nunchuk-coin-control
description: Inspect and manage a wallet's coins (UTXOs) - list and filter coins, lock and unlock them, and organize them with tags and collections including automatic rules (add untagged coins, add by tag, auto-lock). Use when the user wants to label, organize, protect, or audit individual coins, or set up automatic coin classification.
---

# Nunchuk Coin Control

If auth or network setup is the blocker, use `nunchuk-setup`.

To spend specific coins, spend from a tag, or set change-coin tags, use `nunchuk-wallet-transactions` (`tx create --coin` / `--from-tag` / `--change-tags`).

## List Coins

List a wallet's coins with status, amount, received date-time, lock state, tags, and collections:
```bash
nunchuk coin list --wallet <wallet-id>
```

Filter the listing:
```bash
nunchuk coin list --wallet <wallet-id> --status CONFIRMED
nunchuk coin list --wallet <wallet-id> --tag kyc
nunchuk coin list --wallet <wallet-id> --untagged
nunchuk coin list --wallet <wallet-id> --collection "Exchange A"
```

`--status` takes one of:
- `CONFIRMED` — unspent and confirmed on-chain
- `INCOMING_PENDING_CONFIRMATION` — received but not yet confirmed
- `OUTGOING_PENDING_SIGNATURES` — reserved as an input of a pending transaction that still needs signatures
- `OUTGOING_PENDING_BROADCAST` — reserved as an input of a fully signed transaction awaiting broadcast

Each coin's outpoint (`txid:vout`) is what `--coin` options take everywhere (`coin lock`, `coin tag add`, `tx create --coin`, ...).

## Lock and Unlock

A locked coin is never picked by automatic coin selection or `--send-all` sweeps. Explicitly selecting it with `tx create --coin` still spends it — the lock guards against accidental use, not deliberate use.

```bash
nunchuk coin lock --wallet <wallet-id> --coin <txid>:<vout>
nunchuk coin unlock --wallet <wallet-id> --coin <txid>:<vout>
```

`--coin` is repeatable to lock or unlock several coins at once.

## Tags

Tags are reusable labels for classifying coins (for example `kyc`, `exchange-a`, `payroll`). Names are case-sensitive, contain no whitespace, and a leading `#` is accepted and stripped.

```bash
nunchuk coin tag create kyc --wallet <wallet-id>
nunchuk coin tag list --wallet <wallet-id>
nunchuk coin tag add kyc --wallet <wallet-id> --coin <txid>:<vout> --coin <txid>:<vout>
nunchuk coin tag remove kyc --wallet <wallet-id> --coin <txid>:<vout>
nunchuk coin tag rename kyc --wallet <wallet-id> --name kyc-verified
nunchuk coin tag delete kyc --wallet <wallet-id>
```

Show a tag's member coins with live amounts and the spendable total:
```bash
nunchuk coin tag get kyc --wallet <wallet-id>
```

## Collections

Collections are named groups of coins with optional auto-membership rules. Names are case-sensitive, unique per wallet, and may contain spaces (quote them).

```bash
nunchuk coin collection create "Exchange A" --wallet <wallet-id>
nunchuk coin collection list --wallet <wallet-id>
nunchuk coin collection get "Exchange A" --wallet <wallet-id>
nunchuk coin collection add "Exchange A" --wallet <wallet-id> --coin <txid>:<vout>
nunchuk coin collection remove "Exchange A" --wallet <wallet-id> --coin <txid>:<vout>
nunchuk coin collection delete "Exchange A" --wallet <wallet-id>
```

Rule flags on `create` (and `update`):
- `--add-untagged` — new coins arriving without tags join automatically (`--no-add-untagged` to turn off on `update`)
- `--add-tag <tag>` — coins join when they receive one of these tags (repeatable; `--clear-add-tags` to reset on `update`)
- `--auto-lock` — coins are locked the moment they join (`--no-auto-lock` to turn off on `update`)
- `--apply-to-existing` — one-shot: run the rules over currently known coins now

Quarantine posture — lock every unclassified inbound coin until reviewed:
```bash
nunchuk coin collection create quarantine --wallet <wallet-id> --add-untagged --auto-lock
```

Auto-collect coins as they get tagged:
```bash
nunchuk coin collection create verified --wallet <wallet-id> --add-tag kyc --apply-to-existing
```

Update rules later (members are kept):
```bash
nunchuk coin collection update quarantine --wallet <wallet-id> --no-auto-lock
nunchuk coin collection update verified --wallet <wallet-id> --add-tag cold
```

## Change-Coin Tags

When a transaction produces change from tagged inputs, the change coin inherits the inputs' tags by default. Override at creation with `tx create --change-tags <subset|none>` (see `nunchuk-wallet-transactions`). The choice is applied automatically when the change coin is next seen — even if the transaction was broadcast by the mobile app or backend, not the CLI.

## Common requests

User asks: which coins do I have, and which are locked or tagged:
```bash
nunchuk coin list --wallet <wallet-id>
```

User asks: protect a coin from being spent:
```bash
nunchuk coin lock --wallet <wallet-id> --coin <txid>:<vout>
```

User asks: tag my exchange coins:
```bash
nunchuk coin tag create exchange-a --wallet <wallet-id>
nunchuk coin tag add exchange-a --wallet <wallet-id> --coin <txid>:<vout> --coin <txid>:<vout>
```

User asks: how much do my KYC coins add up to:
```bash
nunchuk coin tag get kyc --wallet <wallet-id>
```

User asks: lock every new coin until I review it:
```bash
nunchuk coin collection create quarantine --wallet <wallet-id> --add-untagged --auto-lock
```

## Defaults

- Coin-control data (locks, tags, collections) is stored **locally and encrypted, per machine** — it does not sync with the Nunchuk mobile app or server.
- Collection rules are evaluated lazily when the CLI next sees the wallet's coins (`coin list`, `tx create`, `tx draft`, `tx broadcast`) — there is no background process.
- `coin tag get` / `coin collection get` join local membership with the live unspent set: spent members are marked `[spent]` and the `Total:` line sums spendable members only.
- Use `--json` when the user needs exact machine-readable output.

## Gotchas

- A lock only guards **automatic** selection — `tx create --coin` spends a locked coin.
- Collection membership applies on arrival and is never re-evaluated or auto-removed: removing a tag keeps the coin in collections it already joined, and deleting a collection keeps its coins' lock state.
- Auto-lock fires once, on join — unlocking a member coin afterwards sticks.
- A change coin that inherits tags does **not** join `--add-untagged` collections (it is not untagged when the rules run).
- Tag and collection names are case-sensitive (`KYC` ≠ `kyc`); a lookup miss suggests near-matches.
- `--untagged` cannot be combined with `--tag`.
- Deleting a tag removes it from every coin, collection rule, and pending change-tag intent.
