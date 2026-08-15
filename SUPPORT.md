# GhostKey Support

Thanks for using GhostKey. This page covers how to get help and the fixes for
the most common issues.

## Getting help

- **Bugs & feature requests:** open an issue → https://github.com/wushu75/Ghostkey/issues
- **Questions & ideas:** start a discussion → https://github.com/wushu75/Ghostkey/discussions
- **Documentation:** the [README](README.md) covers install, the integration protocol, and the security model.

When filing a bug, please include: your browser and version, GhostKey version
(see `manifest.json` / the extension card), what you did, what you expected, and
what happened. If it's a service-worker error, copy it from the **Errors** panel
on `chrome://extensions` (or the service-worker DevTools console).

## Troubleshooting

### I forgot my master passphrase
It cannot be recovered — by design, GhostKey never stores or transmits it, and
without it the encrypted vault can't be decrypted. Your options are to restore
from an encrypted backup you exported earlier (you'll still need the passphrase
that backup was created with), or to reset. To reset, remove the extension's
stored data (uninstall and reinstall, or clear its storage) and set up a new
vault. Going forward, keep a backup export and record your passphrase in a
password manager.

### "Manifest file is missing or unreadable" when loading unpacked
The folder you selected must contain `manifest.json` at its **top level**. If you
double-clicked the zip, the files may be nested one folder deeper — select that
inner folder, or re-extract so `manifest.json`, `icons/`, and `src/` sit directly
in the folder you point Chrome at.

### The vault re-locks whenever I close the side panel
This is expected. For security, the decrypted key lives only in the service
worker's memory while unlocked, and Chrome may shut the worker down when nothing
is open. If you'd like it to survive short restarts, enable **Settings → Keep
unlocked this session** (a documented trade-off: the key is cached in
`chrome.storage.session`, in memory, until the browser closes).

### A proxied request fails with a network/CORS error
GhostKey can only reach provider APIs it has host permission for. The built-in
providers (OpenAI, Anthropic, Gemini, xAI, OpenRouter, Groq, DeepSeek, Mistral,
Qwen, Moonshot) and `localhost` work out of the box. If you configured a
**custom** endpoint, GhostKey will ask for permission for that origin the first
time it's used — approve it. If a provider still refuses, double-check the base
URL and that your key is valid for that provider.

### Anthropic requests fail from an extension
Anthropic requires a browser-access header for direct calls; GhostKey sends it
automatically. If you still see an error, confirm your key is active and has
credit, and that you're using a current model name.

### A key won't save / I see a service-worker error
First make sure the vault is **unlocked** (the top chip should read "Unlocked").
If you saw a one-time error right after first-run, update to the latest build —
an early version could log a harmless background warning. If it persists, grab
the exact text from the Errors panel and open an issue.

### Auto-lock is too aggressive / not locking
Adjust **Settings → Auto-lock** (minutes; `0` disables it). The timer resets on
activity, so it counts idle time, not total time.

### Usage or cost looks wrong
Cost figures are **estimates** from a built-in price table and may lag provider
pricing changes. They're meant for awareness and caps, not billing. The
authoritative numbers are always in your provider dashboard.

### Provider isn't listed
Use the **Custom (OpenAI-compatible)** option with the provider's base URL, or
open an issue to request it — most providers are a one-line addition.

## Security issues

Please do **not** file public issues for security vulnerabilities. Report them
privately through the repository's security advisory page
(https://github.com/wushu75/Ghostkey/security/advisories/new) so they can be
fixed before disclosure. GhostKey is local-first with no servers, but responsible
disclosure of any client-side issue is greatly appreciated.

## Before you report

- You're on the latest version.
- You've reloaded the extension on `chrome://extensions` after updating.
- You can reproduce the issue in a clean profile (rules out conflicts).
