# Coldcard HSM Spending Rules

Source: https://coldcard.com/docs/hsm/rules/
Fetched: 2026-04-09

## HSM Policy JSON File

The HSM policy is established by a simple JSON file, which configures various security settings, and the rules for which transactions should be signed automatically by the Coldcard. This JSON file is uploaded to the Coldcard, which parses it, and creates a text version of the policy for approval on-screen.

The file has two parts: global settings and a variable number of `rules`.

Top-level values:
- `notes`: free-form text shown at top of confirmation message, up to 80 chars
- `period`: velocity period in minutes, shared across all rules
- `must_log`: fail anything that cannot be logged to SD card
- `never_log`: disable all log generation, even if SD card is inserted
- `warnings_ok`: allow PSBT signing even when warnings are present
- `msg_paths`: enable message signing only for listed derivation paths; use `any` to allow any path
- `share_xpubs`: enable xpub sharing for listed paths; `m` is always shared regardless
- `share_addrs`: enable address sharing for listed paths; `p2sh` also allows p2sh address calculations
- `set_sl`: load the storage locker with a text value
- `allow_sl`: number of times the storage locker can be read per boot-up
- `boot_to_hsm`: six-digit numeric code used to escape boot-to-HSM
- `priv_over_ux`: reduce chattiness of HSM status responses
- `rules`: list of rule objects; checked in order

Each rule may include:
- `whitelist`: destination addresses allowed by the rule
- `per_period`: total BTC that can move through the rule in the period
- `max_amount`: maximum BTC per transaction for the rule
- `users`: usernames allowed to approve
- `min_users`: number of listed users required; if omitted, all listed users are required
- `local_conf`: require local 6-digit keypad approval
- `wallet`: multisig wallet name, or `"1"` for non-multisig only

When a rule element is missing or `null`, that restriction does not apply.

If no rules are defined, then no PSBT will be signed. An empty rule object (`{}`) allows any transaction to be signed.

## Spending Period (Velocity)

To implement spending limits based on time, Coldcard requires a `period`, expressed in minutes, and it applies to all rules that define `per_period`.

The period starts when it is first used. There is no absolute concept of time on the device because there is no real-time clock. There is only one period. It begins as soon as any rule using `per_period` is applied successfully. At the end of the period, totals reset to zero.

## Spending Rules

Multiple spending rules can be defined. The system scans rules from first to last. The first rule that is satisfied is applied, and later rules are not considered.

Recommendation: place narrow rules first and broader catch-all rules later.

### Max Transaction Amount

`max_amount` limits a single transaction, but repeated smaller transactions can work around it. It is more useful when combined with other controls such as `local_conf`.

### Per-Period Limit

`per_period` is the total amount, in satoshis, that can be spent using that rule during the active period.

### Authorizing Users

The `users` field lists usernames that can approve. If defined, `min_users` controls how many are required. If `min_users` is omitted, all listed users must confirm.

The `local_conf` boolean enables the local confirmation code and requires it for transactions that use the rule.

### Limit to Named Wallet

The `wallet` field can be omitted, set to a multisig wallet name, or set to `"1"` to apply only to the non-multisig wallet.

### Whitelist Address

The `whitelist` field restricts rules to PSBTs whose destination addresses are all in the whitelist.

## Global Policy Values

### Logging to MicroSD Card

`must_log` and `never_log` control logging behavior. By default, Coldcard logs if a card is inserted, but does not fail if the card is missing. `must_log` forces failure if logging is unavailable. `never_log` disables logging entirely.

If `ckcc sign` fails with `OSError: read error` when no SD card is inserted, set `never_log` to `true` in `policy.json`.

### Warnings Okay?

`warnings_ok` allows signing PSBT files that have warnings, such as large fees or unusual derivation paths.

### Message Signing

To enable text message signing, list one or more derivation paths in `msg_paths`. The special value `"any"` allows any path. A star in the final position allows any value in that position only, for example:
- `m/84'/0'/0/*`
- `m/84'/0'/0'/*'`
- `m/9984/*`

### Sharing Xpubs

If `share_xpubs` is defined, Coldcard can calculate xpub values for derived paths. The master xpub `m` is always available over USB and cannot be disabled.

### Sharing Addresses

If `share_addrs` is defined, Coldcard can calculate wallet addresses for whitelisted derivation paths. Star patterns, `any`, and `p2sh` can be used.

In multisig wallets, the script provided is not checked beyond the normal checks for inclusion in a known multisig wallet.

### Storage Locker

The storage locker is a small number of bytes held in the secure element. `set_sl` can write a text value into it. The value is protected by the master PIN and has similar protection to the master seed.

The locker can be read over USB in HSM mode, but reads are limited by `allow_sl`. CKBunker uses the storage locker to store a 32-byte secret used to unlock its NaCL secret box.

### Boot-to-HSM

If `boot_to_hsm` is defined, Coldcard starts in HSM mode immediately after boot and entry of the master PIN.

If `boot_to_hsm` is a 6-digit numeric code, entering that code within the first 60 seconds leaves HSM mode.

If `boot_to_hsm` is set to a non-numeric value that cannot be entered on the keypad, Coldcard can never leave HSM mode.

Bricking hazard:
- no changes to firmware, HSM policy, or settings will be possible again
- not even the master PIN holder can change HSM policy or escape HSM mode
- firmware upgrades are not possible

### Privacy over UX

If `priv_over_ux` is true, HSM status responses over USB will not share:
- text summary of the policy
- approval / refusal counts
- storage locker read count
- period length
- period end time
- per-rule spending in current period
- system uptime
- usernames
- number of users who have provided auth credentials for the current PSBT
