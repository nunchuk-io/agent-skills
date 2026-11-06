---
name: nunchuk-setup
description: Install Nunchuk CLI, authenticate, switch between mainnet and testnet, change Electrum server, set the default fee level, and inspect saved config. Use when a command needs login, network, or Electrum setup, or when the user asks to check current auth or config.
---

# Nunchuk Setup

## Install

```bash
npm install -g nunchuk-cli
```

Make sure `nunchuk` is on `PATH`.

## Authenticate

Get an API key from https://developer.nunchuk.io/ first.

The API key is a secret: it grants full access to the user's Nunchuk account. It must never appear on a command line, in the conversation, or in the agent's output. Login is a step the **user** performs, not the agent.

1. Ask the user to run the login in their own terminal and type the key at the masked prompt. If the agent runtime offers a way for the user to run a command themselves without the agent seeing its input, point them to it.
   ```bash
   nunchuk auth login
   ```
2. Verify from the agent side:
   ```bash
   nunchuk auth status
   ```

Never run `nunchuk auth login --api-key <key>` and never ask the user to paste the key into the chat. Command-line arguments are visible to every process on the machine (`ps`, `/proc/<pid>/cmdline`), are kept in shell history, and are recorded verbatim in agent transcripts.

Unattended use (CI, scheduled jobs): provide `NUNCHUK_API_KEY` in the environment from a secrets manager. From `nunchuk-cli` 0.2.3 every command, including `nunchuk auth login`, honours it, so the one-time login can run without a keyboard. On 0.2.2 `auth login` must be done once at a terminal; the variable then overrides the stored key for the other commands. Do not write the key inline on the command line (`NUNCHUK_API_KEY=... nunchuk ...`), which exposes it the same way a flag does.

Operators who want every irreversible step to be explicit in the command itself can set `NUNCHUK_REQUIRE_YES=1` (nunchuk-cli >= 0.2.3). The CLI then refuses `tx sign` and `tx broadcast` unless `--yes` is passed; see `nunchuk-wallet-transactions` for when the agent may pass it.

## Configure Network

Set network:
```bash
nunchuk network set testnet
```

Network values are `mainnet` or `testnet`.

## Configure Electrum

Show the active Electrum server:
```bash
nunchuk config electrum get
```

Set a custom Electrum server:
```bash
nunchuk config electrum set ssl://electrum.example.com:50002
```

Reset to the network default:
```bash
nunchuk config electrum reset
```

## Configure Default Fee Level

Set the account's default fee level for `tx create` (`economy`, `standard`, or `priority`):
```bash
nunchuk config fee-rate get
nunchuk config fee-rate set priority
nunchuk config fee-rate reset
```

See `nunchuk-wallet-transactions` for per-transaction fee options.

## Inspect

Check auth status:
```bash
nunchuk auth status
```

Inspect saved config/network:
```bash
nunchuk config show
```

## Versions

Check the installed version with `nunchuk --version`. These need `nunchuk-cli >= 0.2.3`: `auth login` reading `NUNCHUK_API_KEY`, `auth login --api-key-stdin`, `key import` with a masked prompt or `--stdin` / `--file`, the `key generate` output guard, and `--yes` on `tx sign` / `tx broadcast`. Batch payments and `tx import` need `>= 0.2.2`; Miniscript needs `>= 0.1.2`.
