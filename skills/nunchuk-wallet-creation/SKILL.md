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
- [ ] Finalize when every required slot is filled and synced
- [ ] Get a fresh receive address if needed
- [ ] Export a wallet backup if needed

If the user specifically asks about invitations, use `nunchuk-invitations`.

If the user specifically asks about Platform key behavior or policy updates, also use `nunchuk-platform-key`.

If the user specifically asks about wallet backup or recovery material, also use `nunchuk-wallet-management`.

If the user asks to create a wallet:
- If they want to continue an existing setup, check `nunchuk sandbox list` first.
- Do not inspect auth status, config, network, or existing finalized wallets first unless the user asked or create fails.
- If they did not specify the wallet structure, ask whether they want:
  - plain multisig like `2-of-3`, `2-of-4`, etc
  - a Miniscript policy with fallback keys, timelocks, or hash-preimage branches
- Ask whether they want Platform key policies such as spending limits, signing delay, or auto-broadcast.
- Ask whether they want an extra key for recovery or robustness.
- When adding keys, ask whether they want to use a key on this device or add a key from Nunchuk mobile/desktop app, another signer, or a hardware wallet.
- If the user did not specify an address type, use `NATIVE_SEGWIT`.
- If the user did not provide a name, use a simple placeholder like `"My Wallet"`.

## Multisig create

Create a plain multisig sandbox:
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

## Miniscript create

If the user asks which Miniscript path is satisfiable, what a signing path means, or how a branch will sign, also use `nunchuk-wallet-transactions`.

Read `references/bip-0379.md` when you need more Miniscript background, extra fragment patterns, or terminology.

Miniscript sandboxes currently support `NATIVE_SEGWIT` only.

Create a Miniscript sandbox:
```bash
nunchuk sandbox create --name "My Wallet" \
  --miniscript-template "multi(2,key_0_0,key_1_0,key_2_0)" \
  --address-type NATIVE_SEGWIT
```

## Add keys

Generate a software key on this device and add it by fingerprint to a multisig slot:
```bash
nunchuk key generate --name "My key"
nunchuk sandbox add-key <sandbox-id> --slot 0 --fingerprint <xfp>
```

Reuse an existing software key on this device:
```bash
nunchuk key list
nunchuk sandbox add-key <sandbox-id> --slot 0 --fingerprint <xfp>
```

Add a local software key to a Miniscript slot:
```bash
nunchuk key list
nunchuk sandbox add-key <sandbox-id> --slot key_0_0 --fingerprint <xfp>
```

Add key with structured fields:
```bash
nunchuk sandbox add-key <sandbox-id> --slot 0 \
  --fingerprint "1a2b3c4d" \
  --xpub "xpub..." \
  --path "m/48h/0h/0h/2h"
```

Add key with a descriptor:
```bash
nunchuk sandbox add-key <sandbox-id> --slot key_1_0 \
  --descriptor "[1a2b3c4d/48h/0h/0h/2h]xpub..."
```

For Miniscript, `--slot` uses signer names from the template such as `key_0_0` or `key_3_0`.

## Check, sync, and finalize

Check sandbox state:
```bash
nunchuk sandbox get <sandbox-id>
```

Enable platform key for multisig:
```bash
nunchuk sandbox platform-key enable <sandbox-id>
```

Enable platform key for Miniscript:
```bash
nunchuk sandbox platform-key enable <sandbox-id> --slot key_3_0
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
nunchuk sandbox finalize <sandbox-id>
nunchuk wallet address get <wallet-id>
nunchuk wallet export <wallet-id> > wallet-backup.txt
```

Important sync states:
- `ADDED` means a slot is known filled remotely, but this device has not synced the full signer descriptor yet.
- `sandbox get` can sync encrypted sandbox state for other participants.
- If finalize says other devices need to come online briefly, run `nunchuk sandbox get <sandbox-id>` and retry after the other devices sync.
- If `sandbox get` shows `status: ACTIVE`, run `nunchuk sandbox finalize <sandbox-id>` to create the local wallet record.

## Examples

Create a plain 2-of-3 multisig wallet:
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

Create a Miniscript wallet using one of the templates from `Miniscript template examples` below:
```bash
nunchuk sandbox create --name "My Wallet" \
  --miniscript-template "multi(2,key_0_0,key_1_0,key_2_0)" \
  --address-type NATIVE_SEGWIT
```

Create a Miniscript 2-of-3 wallet with a 100 USD daily Platform key spending limit:
```bash
nunchuk sandbox create --name "My Wallet" \
  --miniscript-template "or_d(multi(2,key_0_0,key_1_0,key_2_0),and_v(v:pk(key_3_0),after(1735689600)))" \
  --address-type NATIVE_SEGWIT
nunchuk sandbox platform-key enable <sandbox-id> --slot key_3_0
nunchuk sandbox platform-key set-policy <sandbox-id> \
  --limit-amount 100 --limit-currency USD --limit-interval DAILY
```

## Miniscript template examples

Simple 2-of-3:
```text
multi(2,key_0_0,key_1_0,key_2_0)
```
Meaning:
- Any 2 of `key_0_0`, `key_1_0`, and `key_2_0` can spend at any time.
- There is no timelock branch and no hash-preimage requirement.

2-of-3 with an absolute block-height recovery key:
```text
or_d(multi(2,key_0_0,key_1_0,key_2_0),and_v(v:pk(key_3_0),after(840000)))
```
Meaning:
- The normal path is still `2-of-3` at any time.
- The recovery path lets `key_3_0` spend alone once chain height reaches block `840000`.
- `840000` is a block height, not seconds.

2-of-3 with an absolute timestamp recovery key:
```text
or_d(multi(2,key_0_0,key_1_0,key_2_0),and_v(v:pk(key_3_0),after(1735689600)))
```
Meaning:
- The normal path is still `2-of-3` at any time.
- The recovery path lets `key_3_0` spend alone once chain time reaches UNIX timestamp `1735689600`.
- `1735689600` is `2025-01-01 00:00:00 UTC`.

2-of-3 with a relative block-delay recovery key:
```text
or_d(multi(2,key_0_0,key_1_0,key_2_0),and_v(v:pk(key_3_0),older(144)))
```
Meaning:
- The normal path is still `2-of-3` at any time.
- The recovery path lets `key_3_0` spend alone after the UTXO being spent has aged by `144` blocks.
- At roughly 10 minutes per block, `144` blocks is about 24 hours.

2-of-3 with a relative time-delay recovery key:
```text
or_d(multi(2,key_0_0,key_1_0,key_2_0),and_v(v:pk(key_3_0),older(4194473)))
```
Meaning:
- The normal path is still `2-of-3` at any time.
- The recovery path lets `key_3_0` spend alone after a time-based BIP68 delay.
- `4194473` is not seconds. It is a raw sequence value:
  - `4194304` sets the BIP68 time-based flag
  - the remaining `169` means `169` units of `512` seconds
  - `169 * 512 = 86528` seconds, which is about `24 hours 1 minute 28 seconds`

2-of-3 with a hash-preimage branch:
```text
or_d(multi(2,key_0_0,key_1_0,key_2_0),and_v(v:pk(key_3_0),sha256(2222222222222222222222222222222222222222222222222222222222222222)))
```
Meaning:
- The normal path is still `2-of-3` at any time.
- The alternate path lets `key_3_0` spend alone only if the spender also reveals the 32-byte preimage whose SHA256 matches the given hash.

### Timelock guide

`after(n)` is an absolute timelock:
- it uses Bitcoin's absolute locktime rules
- valid values are `1` through `2147483647` (`2^31 - 1`)
- if `n < 500000000`, Bitcoin interprets it as a block height
- if `n >= 500000000`, Bitcoin interprets it as a UNIX timestamp in seconds
- if `n >= 2147483648`, the value is invalid because it exceeds the `2^31 - 1` limit, even if it looks like a future UNIX timestamp

Examples:
- `after(840000)` means the path is valid once chain height reaches block `840000`; it does not mean `840000` seconds
- `after(1735689600)` means the path is valid once chain time reaches `2025-01-01 00:00:00 UTC`
- `after(2147483647)` is the latest valid timestamp lock and means `2038-01-19 03:14:07 UTC`
- `after(2208988800)` is invalid because `2208988800` means `2040-01-01 00:00:00 UTC`
- For a calendar target beyond `2038-01-19 03:14:07 UTC`, use a height lock instead: get the current height with `nunchuk network tip`, estimate the target height, and use `after(<target-height>)`
- Height locks are approximate calendar time because block intervals vary; estimate with the current height plus roughly `144` blocks per day

`older(n)` is a relative timelock enforced through input sequence:
- it uses Bitcoin's relative locktime rules from `OP_CHECKSEQUENCEVERIFY`
- it starts counting from the confirmation of the UTXO being spent, not from wallet creation
- if `n < 4194304`, the BIP68 time-based flag is not set, so the value is block-based
- if `n >= 4194304`, the value is a raw BIP68 time-based sequence encoding, not plain seconds

Examples:
- `older(144)` means the UTXO must be at least `144` blocks old before that path can be used; at roughly 10 minutes per block, that is about 24 hours
- `older(4194473)` means:
  - subtract the time flag `4194304`, leaving `169`
  - interpret `169` as `169` units of `512` seconds
  - `169 * 512 = 86528` seconds, about `24 hours 1 minute 28 seconds`
- `older(86400)` does not mean one day; for time-based relative locks the number must already be BIP68-encoded

For static signing-path inspection or satisfiability checks, use `nunchuk-wallet-transactions`.

## Defaults

- Use `NATIVE_SEGWIT` by default.
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
- For hardware keys, HWI can help export a descriptor to add here: https://github.com/bitcoin-core/HWI/releases
- Miniscript support requires `nunchuk-cli >= 0.1.2`.
- Miniscript support is currently `NATIVE_SEGWIT` only. Taproot Miniscript is not supported here.
- Finalize only after every required slot is filled and synced.
- Accounts may have a limit on wallets or sandboxes. If creation fails because of that, ask whether to delete unused wallets or sandboxes, and only run delete after the user confirms.
