---
name: nunchuk-platform-key
description: Enable, inspect, disable, and update Nunchuk Platform key policies for wallets. Use when the user asks about Platform key, spending limits, signing delay, auto-broadcast, or policy updates.
---

# Nunchuk Platform Key

If auth or network setup is the blocker, use `nunchuk-setup`.

## What is Platform Key

Platform key is an optional Nunchuk-managed signer that can participate in wallet policy enforcement.
It is a secure HSM key managed by Nunchuk.

For multisig wallets, the Platform key can occupy one signer slot.

For Miniscript wallets, the Platform key can be attached to one or more named signer slots from the Miniscript template.

For example, in a `2-of-3` wallet with two user keys and one Platform key, each transaction still needs 2 signatures. That can be:
- the user's two own keys
- one user key and the Platform key

Whether the Platform key signs depends on the active policy.

## Sandbox

In a sandbox, this is where you enable the Platform key and set the initial policies that will apply after wallet creation.

Enable platform key for multisig:
```bash
nunchuk sandbox platform-key enable <sandbox-id>
```

Enable platform key for Miniscript:
```bash
nunchuk sandbox platform-key enable <sandbox-id> --slot key_3_0
nunchuk sandbox platform-key enable <sandbox-id> --slot key_3_0 --slot key_4_0
```

Disable platform key:
```bash
nunchuk sandbox platform-key disable <sandbox-id>
```

Set a global policy:
```bash
# Auto-broadcast, max 100 USD per day.
nunchuk sandbox platform-key set-policy <sandbox-id> \
  --auto-broadcast \
  --limit-amount 100 --limit-currency USD --limit-interval DAILY
```

Set a per-key policy:
```bash
# Only for signer 1a2b3c4d, with a 24h delay, unlimited spending.
nunchuk sandbox platform-key set-policy <sandbox-id> \
  --signer 1a2b3c4d --signing-delay 24h
```

Set different policies for different keys:
```bash
# Signer 1a2b3c4d: 24h delay, unlimited spending.
nunchuk sandbox platform-key set-policy <sandbox-id> \
  --signer 1a2b3c4d --signing-delay 24h

# Signer 5e6f7a8b: auto-broadcast, max 100 USD per day.
nunchuk sandbox platform-key set-policy <sandbox-id> \
  --signer 5e6f7a8b \
  --auto-broadcast \
  --limit-amount 100 --limit-currency USD --limit-interval DAILY
```

Inspect sandbox Platform key state:
```bash
nunchuk sandbox get <sandbox-id>
```

## Existing wallets

On an existing wallet, Platform key commands let you inspect the current policy or request a policy update after the wallet has already been finalized.

A wallet policy update may create a dummy transaction that needs approval signatures. Use `nunchuk-wallet-management` for `wallet dummy-tx list`, `get`, `sign`, and `cancel`.

Inspect wallet Platform key policy:
```bash
nunchuk wallet platform-key get <wallet-id>
```

Update wallet Platform key policy:
```bash
# Auto-broadcast, max 100 USD per day.
nunchuk wallet platform-key update <wallet-id> \
  --auto-broadcast \
  --limit-amount 100 --limit-currency USD --limit-interval DAILY
```

Update different policies for different keys:
```bash
# Submit all per-key policies in one update request.
nunchuk wallet platform-key update <wallet-id> --policy-json '{
  "signers": [
    {
      "masterFingerprint": "1a2b3c4d",
      "autoBroadcastTransaction": false,
      "signingDelaySeconds": 86400
    },
    {
      "masterFingerprint": "5e6f7a8b",
      "autoBroadcastTransaction": true,
      "spendingLimit": {
        "amount": "100",
        "currency": "USD",
        "interval": "DAILY"
      }
    }
  ]
}'
```

## Policy model

- After the wallet is finalized, Nunchuk can use the Platform key to help approve transactions and enforce spending rules.
- Global policy applies to the whole wallet.
- Per-key policy applies to one key, identified by fingerprint.
- If multiple per-key policies apply, Nunchuk uses the most restrictive applicable policy.
- `--auto-broadcast` means Nunchuk broadcasts the transaction after signing, once the transaction is ready.
- `--signing-delay` means Nunchuk waits before signing the transaction with the Platform key.
- `--signing-delay` accepts plain seconds or short durations like `30s`, `15m`, `24h`, and `7d`.
- Spending limit controls how much can be approved automatically within an interval.
- If no spending limit is set, the policy is unlimited.
- Spending limit requires amount, currency, and interval together.
- Supported spending-limit currencies are `USD`, `BTC`, and `sat` only.
- Global and per-key policies cannot be mixed in the same configuration.

## Gotchas

- For multisig wallets, `sandbox platform-key enable` reserves the last key slot for the Nunchuk-held Platform key.
- For Miniscript wallets, `sandbox platform-key enable` requires at least one named signer slot from the Miniscript template; repeat `--slot` or comma-separate slots for multiple Platform key slots.
- Miniscript slot names come from the wallet template, for example `key_3_0`.
- Miniscript support is currently `NATIVE_SEGWIT` and `TAPROOT`.
- Platform key signing is asynchronous. After user signatures are added, check transaction status again before expecting a Platform key signature or broadcasting.
- `sandbox platform-key set-policy` requires Platform key to be enabled first.
- For existing wallets, submit all per-key policies in one `wallet platform-key update` request. Missing non-platform signers are rejected.
- `wallet platform-key update` may return a dummy transaction with type `UPDATE_PLATFORM_KEY_POLICIES`. If that happens, use `nunchuk-wallet-management` for `wallet dummy-tx get`, `sign`, and `cancel`.
