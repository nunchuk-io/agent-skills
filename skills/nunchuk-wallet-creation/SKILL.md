---
name: nunchuk-wallet-creation
description: Create Nunchuk Bitcoin wallets, add keys, and finalize them. Use when the user wants to create a wallet.
---

# Nunchuk Wallet Creation

## Default workflow

Progress:
- [ ] Create or join a sandbox
- [ ] Invite participants if needed
- [ ] Optionally enable Nunchuk Platform key and set policies
- [ ] Add keys
- [ ] Check the current sandbox state
- [ ] Finalize when every slot is filled
- [ ] Get a fresh receive address if needed
- [ ] Export a wallet backup if needed

If the user specifically asks about invitations, use `nunchuk-invitations`.

If the user specifically asks about Platform key behavior or policy updates, also use `nunchuk-platform-key`.

If the user specifically asks about wallet backup or recovery material, also use `nunchuk-wallet-management`.

If the user asks to create a wallet:
- If the user wants to continue an existing wallet setup, check `nunchuk sandbox list` first.
- Do not inspect auth status, config, network, or existing finalized wallets first unless the user asked for that or the create command fails.
- If the user does not specify the wallet structure, ask what setup they want, such as `2-of-3` or `2-of-4`.
- Ask whether they want Platform key policies such as spending limits, signing delay, or auto-broadcast.
- Ask whether they want an extra key for recovery or robustness.
- When adding keys, ask whether they want to use a key on this device or add a key from Nunchuk mobile/desktop app, another signer, or a hardware wallet.
- If the user did not specify an address type, use `NATIVE_SEGWIT`.
- If the user did not provide a name, use a simple placeholder like `"My Wallet"`.

Create:
```bash
nunchuk sandbox create --name "My Wallet" --m 2 --n 3 --address-type NATIVE_SEGWIT
```

Join:
```bash
nunchuk sandbox join <sandbox-id>
```

Join from URL:
```bash
nunchuk sandbox join https://nunchuk.io/wallet/join/<sandbox-id>
```

Generate a software key on this device and add it by fingerprint:
```bash
nunchuk key generate --name "My key"
nunchuk sandbox add-key <sandbox-id> --slot 0 --fingerprint <xfp>
```

Reuse an existing software key on this device:
```bash
nunchuk key list
nunchuk sandbox add-key <sandbox-id> --slot 0 --fingerprint <xfp>
```

Add key with structured fields:
```bash
nunchuk sandbox add-key <sandbox-id> --slot 0 \
  --xpub "xpub..." \
  --derivation-path "m/48h/0h/0h/2h" \
  --master-fingerprint "1a2b3c4d"
```

Add key with a descriptor:
```bash
nunchuk sandbox add-key <sandbox-id> --slot 1 \
  --descriptor "[1a2b3c4d/48h/0h/0h/2h]xpub..."
```

Check state:
```bash
nunchuk sandbox get <sandbox-id>
```

Enable platform key:
```bash
nunchuk sandbox platform-key enable <sandbox-id>
```

For Platform key behavior and policy examples, also use `nunchuk-platform-key`.

Set a global Platform key policy:
```bash
# Auto-broadcast, max 1000 USD per day.
nunchuk sandbox platform-key set-policy <sandbox-id> \
  --auto-broadcast --limit-amount 1000 --limit-currency USD --limit-interval DAILY
```

Set a per-key Platform key policy:
```bash
# Only for signer 1a2b3c4d, with auto-broadcast, a 1h delay, and unlimited spending.
nunchuk sandbox platform-key set-policy <sandbox-id> \
  --signer 1a2b3c4d --auto-broadcast --signing-delay 1h
```

Finalize:
```bash
# Finalize, then get a fresh receive address and export a backup if needed.
nunchuk sandbox finalize <sandbox-id>
nunchuk wallet address get <wallet-id>
nunchuk wallet export <wallet-id> > wallet-backup.txt
```

## Examples

Create a plain 2-of-3 wallet:
```bash
nunchuk sandbox create --name "My Wallet" --m 2 --n 3 --address-type NATIVE_SEGWIT
```

Create a wallet and use a software key stored on this device:
```bash
nunchuk sandbox create --name "My Wallet" --m 2 --n 3 --address-type NATIVE_SEGWIT
nunchuk key generate
nunchuk sandbox add-key <sandbox-id> --slot 0 --fingerprint <xfp>
```

Create a 2-of-3 wallet with a `100 USD` daily Platform key spending limit:
```bash
nunchuk sandbox create --name "My Wallet" --m 2 --n 3 --address-type NATIVE_SEGWIT
nunchuk sandbox platform-key enable <sandbox-id>
nunchuk sandbox platform-key set-policy <sandbox-id> \
  --limit-amount 100 --limit-currency USD --limit-interval DAILY
```

Create a 2-of-3 wallet with a `100 USD` daily spending limit and auto-broadcast:
```bash
nunchuk sandbox create --name "My Wallet" --m 2 --n 3 --address-type NATIVE_SEGWIT
nunchuk sandbox platform-key enable <sandbox-id>
nunchuk sandbox platform-key set-policy <sandbox-id> \
  --auto-broadcast \
  --limit-amount 100 --limit-currency USD --limit-interval DAILY
```

Create a 2-of-3 wallet with a `24-hour` Platform key signing delay, unlimited spending:
```bash
nunchuk sandbox create --name "My Wallet" --m 2 --n 3 --address-type NATIVE_SEGWIT
nunchuk sandbox platform-key enable <sandbox-id>
nunchuk sandbox platform-key set-policy <sandbox-id> \
  --signing-delay 24h
```

## Defaults

- `--address-type` accepts `NATIVE_SEGWIT`, `NESTED_SEGWIT`, `LEGACY`, or `TAPROOT`.
- Use `--json` for raw machine-readable output.
- For plain create requests, prefer a single `sandbox create` command over exploratory checks.
- If the user already has a local software key, use `nunchuk key list` and `sandbox add-key --fingerprint <xfp>` instead of generating a new one.
- If the user wants to use a hot key on this device, generate it with `nunchuk key generate` and add it with `sandbox add-key --fingerprint <xfp>`.
- After finalizing, use `wallet export <wallet-id>` if the user wants a wallet backup.
- Treat `sandbox delete` as destructive and only run it when the user clearly asked to remove the sandbox.

## Gotchas

- Mnemonics, seed phrases, and `xprv` values are critical secrets. Anyone with them can control the funds, and losing them may make the funds unrecoverable.
- Derivation paths may use either `h` or `'`; prefer `h`.
- `key generate` stores the software key on the current device for the current user and network.
- `sandbox add-key --fingerprint <xfp>` is the simplest way to use a locally stored software key.
- For hardware key, HWI can help export a descriptor to add here: https://github.com/bitcoin-core/HWI/releases
- The CLI does not support miniscript wallet yet. Use the Nunchuk app for miniscript flows.
- Accounts may have a limit on wallets or sandboxes. If creation fails because of that, ask whether to delete unused wallets or sandboxes, and only run delete after the user confirms.
- If `sandbox get` shows `status: ACTIVE`, run `nunchuk sandbox finalize <sandbox-id>` to create the local wallet record. This can happen when the wallet was already finalized from another device.
- `TAPROOT` address type is not yet supported.
- Finalize only after every key slot is filled.
