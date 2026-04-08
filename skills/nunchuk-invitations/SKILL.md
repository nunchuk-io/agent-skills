---
name: nunchuk-invitations
description: Send, review, accept, and deny Nunchuk wallet invitations during wallet setup. Use when the user wants to invite others, check whether they were invited, or respond to an invitation.
---

# Nunchuk Invitations

If auth or network setup is the blocker, use `nunchuk-setup`.

If the user is creating a wallet from scratch, also use `nunchuk-wallet-creation`.

## Send invitations

Invite one or more people:
```bash
nunchuk invitation send <sandbox-id> alice@example.com bob@example.com
```

Invite with comma-separated emails:
```bash
nunchuk invitation send <sandbox-id> alice@example.com,bob@example.com
```

## Review invitations

List invitations for the current user:
```bash
nunchuk invitation list
```

List invitations sent for one sandbox:
```bash
nunchuk invitation list <sandbox-id>
```

## Accept or deny

Accept an invitation:
```bash
nunchuk invitation accept <invitation-id>
```

Deny an invitation:
```bash
nunchuk invitation deny <invitation-id>
```

## Defaults

- `invitation send` accepts either space-separated emails or comma-separated batches.
- Use `invitation list` with no sandbox ID for invitations received by the current user.
- Use `invitation list <sandbox-id>` to inspect invitations sent for one sandbox.

## Gotchas

- The logged-in email is invited automatically. To add that user's key, use the Nunchuk mobile app, desktop app, or CLI on that account instead of sending another invitation to the same email.
- Invite other participant email addresses only.
- `invitation accept` joins the sandbox behind the invitation.
- Sandbox-scoped invitation lists do not show invitations sent to your own email.
- `invitation deny` denies the invitation only. It does not delete the sandbox.
