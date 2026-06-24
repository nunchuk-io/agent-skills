---
name: nunchuk-wallet-management
description: Inspect and manage existing Nunchuk wallets, including balances, receive addresses, backup/export, recovery, rename, delete, wallet replacement, and dummy-transaction approval. Use when the user wants to work with a wallet that has already been created.
---

# Nunchuk Wallet Management

If auth or network setup is the blocker, use `nunchuk-setup`.

If the user asks about Platform key inspection or policy updates on an existing wallet, use `nunchuk-platform-key`.

## Inspect

List wallets:
```bash
nunchuk wallet list
```

List wallets without balance lookup (faster, does not require internet):
```bash
nunchuk wallet list --no-balance
```

Get wallet details:
```bash
nunchuk wallet get <wallet-id>
nunchuk wallet get <wallet-id> --no-balance
```

Get a fresh receive address:
```bash
nunchuk wallet address get <wallet-id>
```

## Export and Recover

Export backup material:
```bash
nunchuk wallet export <wallet-id>
```

Export descriptor only:
```bash
nunchuk wallet export <wallet-id> --type descriptor
```

Export descriptor in `all` format:
```bash
nunchuk wallet export <wallet-id> --type descriptor --format all
```

Export BSMS only:
```bash
nunchuk wallet export <wallet-id> --type bsms
```

Save export output to a file:
```bash
nunchuk wallet export <wallet-id> > wallet-backup.txt
nunchuk wallet export <wallet-id> --type bsms > wallet-backup.bsms
```

Recover from a backup file:
```bash
nunchuk wallet recover --file wallet-backup.bsms
```

Recover from a descriptor with a custom name:
```bash
nunchuk wallet recover --file descriptor.txt --name "Recovered Wallet"
```

## Rename and Delete

Rename a wallet:
```bash
nunchuk wallet rename <wallet-id> --name "My Wallet"
```

Delete a wallet:
```bash
nunchuk wallet delete <wallet-id>
```

## Wallet Replacement

Use wallet replacement when the user wants to replace keys or create a replacement sandbox for an existing finalized wallet.

Replacement workflow:
- If starting the replacement, run `wallet replace create <wallet-id>`.
- If another participant started it, run `wallet replace list <wallet-id>` and then `wallet replace accept <wallet-id> <group-id>`.
- Open the replacement sandbox with `sandbox get <group-id>`.
- Add or replace only the changed signer slots with `sandbox add-key`.
- Finalize the replacement with `sandbox finalize <group-id>` once every required slot is filled and synced.
- After finalizing, get a fresh receive address and export a backup if the user wants one.

Create a replacement sandbox:
```bash
nunchuk wallet replace create <wallet-id>
```

List replacement sandboxes:
```bash
nunchuk wallet replace list <wallet-id>
```

Accept and join a replacement sandbox:
```bash
nunchuk wallet replace accept <wallet-id> <group-id>
```

Decline a replacement sandbox locally:
```bash
nunchuk wallet replace decline <wallet-id> <group-id>
```

## Dummy Transactions

Currently dummy transactions are used to approve Platform key policy changes.

They are built as dummy PSBTs that use a dummy input and send back to the same wallet. They are only for collecting the required wallet signatures for approval, and they cannot be broadcast to the blockchain.

List pending dummy transactions:
```bash
nunchuk wallet dummy-tx list <wallet-id>
```

Get one dummy transaction:
```bash
nunchuk wallet dummy-tx get <wallet-id> --dummy-tx-id <id>
```

Sign a dummy transaction:
```bash
nunchuk wallet dummy-tx sign <wallet-id> --dummy-tx-id <id>
```

Cancel a dummy transaction:
```bash
nunchuk wallet dummy-tx cancel <wallet-id> --dummy-tx-id <id>
```

## Defaults

- Prefer `wallet export <wallet-id>` with default options unless the user asked for a specific backup format.
- Use `--json` when the user needs exact machine-readable output.
- Use `--no-balance` for faster local wallet lookup without Electrum balance fetch.
- Treat `wallet delete` as destructive and only run it when the user clearly asked for removal.
- Treat wallet replacement as a sandbox workflow: create or accept the replacement, update signer slots, then finalize.

## Gotchas

- `wallet export` defaults to descriptor export.
- `wallet export` supports `--type descriptor` and `--type bsms`.
- `--format` only applies to descriptor export.
- Export output is raw content, so redirect it to a file when the user wants a backup file.
- `wallet recover` accepts a descriptor file or a BSMS 1.0 backup file.
- `wallet dummy-tx get` shows the request type, old state, new state, and signature progress.
- `wallet dummy-tx sign` can use auto-detected stored keys, `--fingerprint`, or `--xprv`.
- If the recovered wallet already exists locally, the command returns `already_exists` instead of overwriting it.
- `wallet replace create` starts from the current finalized wallet signer set; if the wallet has a Platform key, that Platform key slot is cleared for replacement and the policy is carried into the sandbox.
- `wallet replace accept` joins the replacement sandbox and submits unchanged local wallet signers, so only changed slots normally need new keys.
- `wallet replace decline` records a local decline and does not delete the server-side replacement group.
- `wallet delete` deletes the wallet from the server and removes the local copy.
