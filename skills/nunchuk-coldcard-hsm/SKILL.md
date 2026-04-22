---
name: nunchuk-coldcard-hsm
description: Use Coldcard with Nunchuk, add a Coldcard key, review or create HSM policies, create or authorize HSM users, validate or start HSM mode, and sign PSBTs with Coldcard. Use when the user wants to use Coldcard with Nunchuk, add a Coldcard key to a wallet, or asks about HSM mode.
---

# Nunchuk Coldcard HSM

If the user is creating a wallet, also use `nunchuk-wallet-creation`.

If the user wants to create a wallet with Coldcard or add a Coldcard key, ask whether they want plain Coldcard signing or HSM mode before other Coldcard-specific setup questions, unless they already mentioned HSM.

Requires `ckcc` on `PATH`.

Install `ckcc`:
```bash
pip install 'ckcc-protocol[cli]'
```

## Use Coldcard with Nunchuk

Progress:
- Export the descriptor with `ckcc xpub -v`.
- Add the key to the wallet with `nunchuk sandbox add-key --descriptor`.
- Create the wallet and export the descriptor to a file.
- Enroll it on the Coldcard with `ckcc upload <descriptor-file> --miniscript`.
- Sign a transaction with `nunchuk tx get --wallet <wallet-id> --tx-id <txid> --json | jq -r .psbt > <txid.psbt>`, `ckcc sign --base64 <txid.psbt>`, and `nunchuk tx sign --psbt <signed-psbt-base64>`.

Export a mainnet multisig account descriptor:
```bash
ckcc xpub -v "m/48'/0'/0'/2'"
nunchuk sandbox add-key <sandbox-id> --slot 0 --descriptor "[xfp/48h/0h/0h/2h]xpub..."
```

Export a testnet multisig account descriptor:
```bash
ckcc xpub -v "m/48'/1'/0'/2'"
nunchuk sandbox add-key <sandbox-id> --slot 0 --descriptor "[xfp/48h/1h/0h/2h]tpub..."
```

If you are not sure which network the Coldcard is using:
```bash
ckcc chain
```

Get fingerprint and xpub separately if needed:
```bash
ckcc xfp
ckcc xpub "m/48'/1'/0'/2'"
```

Enroll an exported wallet descriptor:
```bash
nunchuk wallet export <wallet-id> --type descriptor > wallet.txt
ckcc upload wallet.txt --miniscript
```

For MK3 classic multisig config only:
```bash
nunchuk wallet export <wallet-id> --type multisig-config > wallet.txt
ckcc upload wallet.txt --multisig
```

List registered miniscript wallet names:
```bash
ckcc miniscript ls
```

Inspect one registered miniscript wallet:
```bash
ckcc miniscript get <wallet-name>
```

## Use Coldcard in HSM Mode

HSM Mode is the Coldcard feature for signing transactions without a person physically present. It is an advanced setup that requires extra software and a `policy.json` file. Not supported on COLDCARD Q.

Read these references as needed:
- Read `references/coldcard-hsm-rules.md` when creating or reviewing `policy.json` rules and top-level policy settings.
- Read `references/coldcard-hsm-local-codes.md` when the policy uses `local_conf`.
- Read `references/coldcard-hsm-security.md` before enabling `boot_to_hsm`, `priv_over_ux`, or other unattended-signing security settings.

Progress:
- Create, finalize, and enroll the wallet on Coldcard.
- Do not treat the step above as completion for an HSM request.
- Ask whether the user wants unrestricted signing or a custom policy.
- Use unrestricted signing by default if the user does not ask for spending limits, whitelists, local confirmation, or HSM users.
- Create HSM users only if the policy requires user authorization.
- Validate `policy.json` with `ckcc hsm-start -n policy.json`.
- Start HSM Mode with `ckcc hsm-start`.
- Confirm the active policy on the Coldcard screen.
- Check status with `ckcc hsm`.
- Authorize transactions with `ckcc auth <user>` for TOTP or `ckcc auth <user> -p` for password if the active policy requires user authorization.
- Sign transactions with `ckcc sign --base64 <tx.psbt>` so the result can be passed to `nunchuk tx sign --psbt`.

Unrestricted `policy.json`:
```json
{
  "never_log": true,
  "rules": [
    {}
  ]
}
```

Example limited `policy.json`:
```json
{
  "notes": "Office wallet policy",
  "never_log": true,
  "period": 1440,
  "rules": [
    {
      "max_amount": 1000000,
      "per_period": 5000000
    }
  ]
}
```

This example allows up to `0.01 BTC` per transaction and `0.05 BTC` per 24 hours.

For a custom policy, read `references/coldcard-hsm-rules.md` first.

## HSM users

For TOTP, the usual flow is to scan the QR shown on the Coldcard with any RFC6238-compatible authentication app, such as FreeOTP (`https://freeotp.github.io/`) or Google Authenticator. Use `--show-qr` if you want the QR contents locally instead.

Create a TOTP user:
```bash
ckcc user alice
```

Create a password user with a Coldcard-generated password:
```bash
ckcc user alice --pass
```

Create a password user with a password you enter:
```bash
ckcc user alice --ask-pass
```

Create a password user non-interactively:
```bash
ckcc user alice --text-secret "my-secret-password"
```

Authorize a TOTP user for HSM signing:
```bash
ckcc auth alice 123456
```

Authorize a password user for HSM signing:
```bash
ckcc auth alice "my-secret-password" --psbt-file tx.psbt
```

Delete a user:
```bash
ckcc user alice --delete
```

## Start HSM mode

Validate a policy file first:
```bash
ckcc hsm-start -n policy.json
```

Start HSM mode:
```bash
# Confirm on the Coldcard screen.
ckcc hsm-start policy.json
```

Check current HSM status:
```bash
ckcc hsm
```

## Defaults

- Prefer `ckcc xpub -v` because it prints `[xfp/path]xpub` in a form you can paste into `nunchuk sandbox add-key --descriptor`.
- Use unrestricted signing (`never_log: true`, `rules: [{}]`) by default unless the user asks for custom constraints.
- Use `ckcc hsm-start -n` before uploading a new policy file.
- Create HSM users only before activating a policy that references them.

## Gotchas

- If the Coldcard network is wrong, the exported path and `xpub` or `tpub` prefix will not match the wallet network. Check `ckcc chain` if you are unsure.
- Default to `ckcc upload <descriptor-file> --miniscript` for exported descriptors. Use `--multisig` only for MK3 classic multisig config files.
- For signing, enroll the wallet on the Coldcard before entering HSM Mode. If the wallet is unknown, Coldcard may try to import it from the PSBT, but HSM Mode blocks that path with `MS enroll not allowed in HSM mode`.
- Run `ckcc auth` when the active HSM rule requires user authorization. For password users, the approval is tied to the exact PSBT file. For the current PSBT, approvals can be queued before signing.
- `ckcc hsm-start` needs approval on the Coldcard screen.
- `ckcc hsm` shows only a summary of the active policy.
- Use `ckcc sign --base64` for Nunchuk import; without it, `ckcc sign` writes binary PSBT output.
- If `ckcc sign` fails with `OSError: read error` and no SD card is inserted, set `"never_log": true` in `policy.json`.
- Removing a user can invalidate a policy that refers to that user.
- Read `references/coldcard-hsm-rules.md` before editing or reviewing a Coldcard HSM policy file.
- If `policy.json` needs a wallet name for a miniscript wallet, enroll first and use `ckcc miniscript ls` to find the registered wallet name.
- Read `references/coldcard-hsm-local-codes.md` before using `local_conf`.
- Read `references/coldcard-hsm-security.md` before using `boot_to_hsm` or `priv_over_ux`.
