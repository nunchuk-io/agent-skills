# Coldcard HSM Security Notes

Source: https://coldcard.com/docs/hsm/security/
Fetched: 2026-04-09

This section documents the Coldcard team's thinking on operating Coldcard in HSM mode and using CKBunker.

## Human Required to Start HSM Mode

Entry into HSM mode must always require user confirmation on the device screen. Otherwise a desktop program could upload a bogus policy, start HSM mode, and steal funds.

## Privacy over UX

Coldcard should not reveal too much about the current or proposed HSM policy, because knowing the rules makes them easier to stretch or attack. This led to the `priv_over_ux` setting.

## USB Commands

Some USB commands are inappropriate in HSM mode, such as firmware upgrade. Coldcard uses a fixed whitelist to control this.

## PIN without Authority

If an operator has the PIN but not full spending authority, they might try to reboot, upload a permissive HSM policy, enter HSM mode, and steal funds later.

Coldcard's conclusion is to assume anyone with the PIN has full authority over funds, while also supporting `boot_to_hsm` to lock operation into HSM mode.

The 60-second timeout on entering the escape code exists so a local operator who learns the escape code but does not know the master PIN cannot escape HSM mode.

## User Names

A policy using user accounts that do not exist is invalid. Deleting a user can affect HSM mode, so Coldcard does not allow that during operation.

## SHA1 in TOTP

SHA1 is used inside the TOTP implementation because Google Authenticator supports it. FreeOTP supports SHA256, but is less widely used, and the QR code does not have room to indicate alternative hashing algorithms.

## Data Oversharing

Questions considered:
- should XPUB USB commands be allowed?
- should address/P2SH address commands be allowed in HSM mode?

Coldcard's choices:
- xpub and address commands are disabled by default, but can be enabled as needed
- path sharing should be limited so less information is revealed
- the master xpub cannot be disabled because it is used to protect USB communications privacy against MitM attacks

Operational recommendation: do not use the master xpub directly. Use a derived path instead.

## Brute Forcing Policy Rules

Coldcard performs a secure logout if there are more than 100 refused transactions. This is intended as a main defense against brute-forcing the policy rules.

## Racing Between Users

There is a potential race condition between local confirmation and multiple remote users. To address this, the local confirmation code is a function of:
- the PSBT being approved
- a random salt chosen by the Coldcard

The effective PIN is based on HMAC over the PSBT hash with bytes from the Coldcard, then reduced modulo 1,000,000.

## Signing Reveals Pubkeys

`share_addrs` is effectively a superset of `msg_paths`, because signing a message reveals the public key for that path, and the payment address can be derived from that public key.

## HMAC Passwords Backup Limitation

HMAC-based passwords do not survive backup and restore onto a different Coldcard, because the original serial number is used during key stretching.

TOTP users do not have the same issue because the shared secret is stored directly. Coldcard recommends using TOTP instead of passwords, or resetting user passwords after restore.

## HSM Enabled Before USB Enabled

If the attached computer is fully compromised, it could try to send USB commands immediately after unlock and before HSM mode starts.

That is why Coldcard prompts for HSM mode at power-up when a policy file is detected. The USB port is not yet enabled at that point, so there is no window for unprotected USB commands if HSM mode is entered.

`boot_to_hsm` addresses the same concern by making the answer effectively always yes.

## Untrusted Remote Hands

When `boot_to_hsm` is enabled and there are per-period limits, Coldcard starts with the current period active and all maximum amounts already spent. After boot, operators must wait for the velocity period to expire before such rules can be used.

This is meant for cases where remote hands may know the master PIN but should not be able to reset the spending period and bypass velocity limits.

## Side Channel Attacks

Any HSM that performs unattended signing operations may leak key information through side channels. Multiple requests can turn even small leaks into a serious risk over time.

Coldcard notes risk from USB power or data lines even with software countermeasures. Recommended mitigations include:
- placing the Coldcard HSM into a physically secure EMI-shielded box
- isolating USB power and signal
