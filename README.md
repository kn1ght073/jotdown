# jotdown

A tiny single-file notes app with client-side encryption.
**[Try the live demo →](https://kn1ght073.github.io/jotdown/)**

- **Single file.** `index.html` is the whole app — no build step, no server, no dependencies.
- **Local-first.** Notes are stored in your browser's `localStorage`. Nothing is uploaded anywhere.
- **Encrypted snapshots.** Export/import your notes (or arbitrary files) as AES-256-GCM encrypted snapshots.

## Usage

Open `index.html` (or the demo link) and start typing. Click **New note**, edit, and it autosaves.
Use **Export notes as snapshot** to produce an encrypted `.jdn` file you can move between browsers
or machines — open it later with **Open a snapshot** and the same passphrase.

A bundled example snapshot ships inside the app so the import flow can be tried end-to-end;
it is locked with a passphrase that is not published.

## Snapshot format

A snapshot is a small JSON envelope:

```json
{
  "v": 1,
  "kdf": { "name": "PBKDF2-SHA256", "iterations": 600000, "salt_b64": "…" },
  "cipher": "AES-256-GCM",
  "nonce_b64": "…",
  "ct_b64": "…"
}
```

The payload is sealed with AES-256-GCM (authenticated encryption). The key is derived from the
passphrase with PBKDF2-HMAC-SHA256, 600,000 iterations, and a fresh random 128-bit salt per
snapshot; a fresh random 96-bit nonce is used for every encryption. A wrong passphrase fails the
GCM authentication check, so the app learns nothing beyond "right or wrong".

## Notes on the crypto

- Everything happens in the page via the [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API); the app makes no network calls.
- Passphrases are never stored and never leave the browser.
- 600k PBKDF2 iterations make each guess cost roughly half a second of CPU. That matters because
  snapshots are portable: assume an attacker who obtains one can try passphrases offline at full
  speed. Use a long, random passphrase (a password manager or diceware words).
- This is a weekend project, not a security product. For serious secrets use a proper password
  manager or an audited tool.

## Limitations

- `localStorage` is per-browser and per-origin — clear it and your notes are gone. Keep snapshots
  of anything you care about.
- No sync, no sharing, no rich text.

## License

MIT — see [LICENSE](LICENSE).
