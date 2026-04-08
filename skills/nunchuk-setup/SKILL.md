---
name: nunchuk-setup
description: Install Nunchuk CLI, authenticate, switch between mainnet and testnet, change Electrum server, and inspect saved config. Use when a command needs login, network, or Electrum setup, or when the user asks to check current auth or config.
---

# Nunchuk Setup

## Install

```bash
npm install -g nunchuk-cli
```

Make sure `nunchuk` is on `PATH`.

## Authenticate

Get an API key from https://developer.nunchuk.io/ first.

Interactive login:
```bash
nunchuk auth login
```

Non-interactive login:
```bash
nunchuk auth login --api-key <api-secret-key>
```

`auth login` supports both the interactive prompt and `--api-key <api-secret-key>`.

If the agent is executing the command itself, prefer:
```bash
nunchuk auth login --api-key <api-secret-key>
```

Use plain `nunchuk auth login` when the user is running the command manually and will enter the key in the prompt.

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

## Inspect

Check auth status:
```bash
nunchuk auth status
```

Inspect saved config/network:
```bash
nunchuk config show
```
