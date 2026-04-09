# Coldcard HSM Local Confirmation Codes

Source: https://coldcard.com/docs/hsm/local-codes/
Fetched: 2026-04-09

The only interaction possible with a Coldcard in HSM mode is to enter a local authorization code. This can be required by specific HSM policy rules, but is optional.

As the local operator enters the 6-digit numeric code, the digits are shown in the top right corner of the screen. Press `✔` to apply them, or `X` to clear and start over. Codes are always 6 digits.

There is no indication the code worked or failed at entry time, because the code is not checked until the PSBT is later provided for signing.

The required code is derived from:
- the specific bytes of the PSBT being approved
- a salt value picked by the Coldcard

If you already have the PSBT file, `ckcc local-conf` can show the code needed:

```text
% ckcc local-conf debug/attempt.psbt
Local authorization code is:
    160681
```

CKBunker can also reveal the local code.

A different code is required for:
- each attempted signing, because Coldcard changes the salt value
- every PSBT file, because the PSBT hash is part of the calculation

Set `local_conf` in a spending rule to require the correct local code at PSBT approval time.
