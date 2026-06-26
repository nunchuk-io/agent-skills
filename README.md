# Agent Skills

[Agent Skills](https://agentskills.io) for the [Nunchuk CLI](https://github.com/nunchuk-io/nunchuk-cli).

## Installation

Install this repository as a skill pack in your Agent Skills client.

Install all skills for all agents:

```bash
npx skills add nunchuk-io/agent-skills --all --global
```

Or install with custom selection:

```bash
npx skills add nunchuk-io/agent-skills
```

Install `nunchuk` CLI:

```bash
npm install -g nunchuk-cli
```

Quick check:

```bash
nunchuk --help
```

Before using the CLI, get an API key from https://developer.nunchuk.io/ and log in:

```bash
nunchuk auth login
```

## Available Skills

| Skill | Description |
| --- | --- |
| [nunchuk-setup](./skills/nunchuk-setup/SKILL.md) | Install `nunchuk`, log in, switch network, and inspect config |
| [nunchuk-wallet-creation](./skills/nunchuk-wallet-creation/SKILL.md) | Create wallets, add keys, and finalize |
| [nunchuk-invitations](./skills/nunchuk-invitations/SKILL.md) | Invite people to wallets and manage invitations |
| [nunchuk-platform-key](./skills/nunchuk-platform-key/SKILL.md) | Configure and inspect Platform key policies on sandboxes and wallets |
| [nunchuk-coldcard-hsm](./skills/nunchuk-coldcard-hsm/SKILL.md) | Add Coldcard keys, enroll wallets, and sign with Coldcard including HSM mode |
| [nunchuk-wallet-management](./skills/nunchuk-wallet-management/SKILL.md) | List, inspect, export, rename, replace, recover, and delete finalized wallets |
| [nunchuk-wallet-transactions](./skills/nunchuk-wallet-transactions/SKILL.md) | Create, sign, inspect, list, and broadcast wallet transactions |

## Usage

Skills are automatically available once installed. Use them when the user asks how to use the `nunchuk` CLI or wants concrete command examples.

Examples:

```text
Authenticate the nunchuk CLI and switch to testnet
```

```text
Create a 2-of-3 wallet named My Wallet
```

```text
Create a Taproot 2-of-3 wallet
```

```text
Replace a key in my existing wallet
```

```text
Create a 2-of-3 wallet with a 100 USD daily spending limit
```

```text
Create a 2-of-3 wallet with a 100 USD daily spending limit and auto-broadcast
```

```text
Create a 2-of-3 wallet with a 24-hour signing delay
```

```text
Create a wallet with Coldcard in HSM mode
```

```text
Invite Alice and Bob to my wallet
```

```text
Accept a wallet invitation
```

```text
List all my wallets and their balances
```

```text
Do I have any invitations to join wallets?
```

```text
What is the current key policy on my wallet
```

```text
Update my wallet key policy to a 100 USD daily spending limit
```

```text
Show me how to join a sandbox from a full join URL
```

```text
Export a wallet descriptor and BSMS backup
```

```text
Send 100 USD to this address, then sign and broadcast the transaction
```
