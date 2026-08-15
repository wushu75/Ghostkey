<div align="center">

<img src="icons/icon128.png" width="96" height="96" alt="GhostKey" />

# GhostKey

### Your AI keys. One vault. Zero trust required.

**GhostKey** is a local-first, BYOK (Bring Your Own Key) security wallet for AI
API keys. Paste your keys once — they're encrypted and never leave the
extension. Other extensions and web apps request access and receive a
short-lived session token or a **proxied** request. The real key is never
exposed.

**100% client-side · Zero servers · Zero telemetry · Zero accounts**

[Features](#features) · [Install](#installation) · [Integrate](#developer-integration-guide) · [Security](#security-model) · [License](#license)

</div>

---

## Why GhostKey

Pasting raw API keys into random web tools and extensions is how keys leak. Most
"key manager" extensions still hand the raw key to whoever asks. GhostKey flips
the model: keys live in one **encrypted vault**, and everything else gets
**delegated, revocable access** — ideally a proxied request where the key never
leaves the vault at all.

- 🔐 **Encrypted at rest** — AES-256-GCM, PBKDF2 with 600,000 rounds, Web Crypto only.
- 🕶️ **Keys stay hidden** — proxy mode means requesters get responses, never keys.
- ⏱️ **Short-lived sessions** — grants expire and auto-revoke; nothing is permanent by default.
- 🧾 **Full audit log** — every request, grant, deny, and proxy is recorded on-device.
- 💸 **Cost + caps** — estimated spend per provider with a hard monthly kill-switch.
- 🧩 **Simple protocol** — any extension or web app integrates in a few lines.
- 🪶 **No dependencies** — modern ES modules, no build step, easy to audit.

## Features

### Encrypted vault
- Providers: **OpenAI, Anthropic, Google Gemini, xAI/Grok, OpenRouter, Groq,
  DeepSeek, Mistral, Ollama, LM Studio**, and any **custom OpenAI-compatible** endpoint.
- AES-256-GCM authenticated encryption via the Web Crypto API.
- Key derivation: PBKDF2-HMAC-SHA-256, **600,000 iterations**, random 128-bit salt.
- Add / edit / rename / delete / **rotate** keys.
- **Export / import** the vault as an encrypted `.ghostkey` file (still ciphertext — needs your passphrase to open).

### Permission & session system
- Clean external messaging protocol (`GHOSTKEY_*`) for other extensions/apps.
- Beautiful permission prompt: **Allow once / Session / Always / Deny**.
- Short-lived session tokens (configurable, default **30 min**).
- **Request proxying** so the real key never leaves GhostKey.
- Full **audit log**: timestamp, requester, provider, model, status.

### Usage & safety dashboard
- Token usage and **estimated cost** per provider (editable pricing table).
- Hard **monthly spending cap** with a **kill-switch**.
- One-click **Rotate key** and **Revoke all sessions**.
- **Clipboard paste guard** — warns when an API key is pasted into a web page.

### UI
- **Side panel** (full experience) + **popup** (quick access).
- First-run **onboarding** flow.
- Vault list, usage dashboard, sessions, audit log, settings.
- Dark by default, light mode supported, respects reduced motion.

## Installation

### Load unpacked (developers)

1. `git clone https://github.com/ghostkey/ghostkey && cd ghostkey`
2. Open `chrome://extensions` (works in Chrome, Edge, Brave, Arc, Opera…).
3. Toggle **Developer mode** (top-right).
4. Click **Load unpacked** and select the project folder.
5. GhostKey opens an onboarding tab — create your master passphrase.
6. Pin GhostKey from the toolbar puzzle icon for one-click access.

> No build step. No dependencies. What you load is exactly what runs.

## Usage

- **Popup** (toolbar icon): unlock, glance at spend and keys, lock, or open the full panel.
- **Side panel**: manage keys, view usage, sessions, and the audit log, change settings.
- **First key**: side panel → **+ Add key** → pick a provider → paste your key.
- **Backups**: **Export backup** downloads an encrypted `.ghostkey`; **Import backup** restores it with your passphrase.

## Developer integration guide

Other extensions and web apps talk to GhostKey with the standard
`chrome.runtime.sendMessage(<GHOSTKEY_ID>, msg)` external messaging API.
A tiny SDK (`src/lib/ghostkey-client.js`) wraps the protocol.

```js
import { GhostKey } from './ghostkey-client.js';

const gk = new GhostKey('<GHOSTKEY_EXTENSION_ID>'); // from chrome://extensions

// 1) Discover capabilities (never touches keys)
const info = await gk.hello();
// -> { ok, version, providers: [...], locked }

// 2) Ask the user for access (shows the permission prompt)
const grant = await gk.requestAccess('anthropic', {
  appName: 'My Cool App',
  reason: 'Summarize the current page',
});
// -> { ok, token, expiresAt, mode: 'proxy' }

// 3) Proxy a request — GhostKey injects the key and forwards it.
//    You get the response; the key never leaves the vault.
const res = await gk.proxy({
  body: {
    model: 'claude-3-5-haiku-latest',
    max_tokens: 200,
    messages: [{ role: 'user', content: 'Hello!' }],
  },
});
// -> { ok, status, data }

// 4) Optional: end the session early
await gk.release();
```

### Protocol reference

All messages are objects with a `type`. Responses always include `ok`.

| Message | Payload | Response |
| --- | --- | --- |
| `GHOSTKEY_HELLO` | — | `{ ok, version, providers, locked }` |
| `GHOSTKEY_REQUEST_ACCESS` | `{ provider, appName?, reason?, mode? }` | `{ ok, token, expiresAt, mode }` or `{ ok:false, error }` |
| `GHOSTKEY_PROXY` | `{ token, body, path?, method?, headers? }` | `{ ok, status, data }` |
| `GHOSTKEY_REVEAL` | `{ token }` | `{ ok, key, baseUrl }` *(only if reveal granted)* |
| `GHOSTKEY_RELEASE` | `{ token }` | `{ ok }` |

- `mode`: `'proxy'` (default, recommended) or `'reveal'` (requires explicit user opt-in).
- `provider`: `openai`, `anthropic`, `gemini`, `xai`, `openrouter`, `groq`,
  `deepseek`, `mistral`, `ollama`, `lmstudio`, `custom`.
- Common `error` values: `vault_locked`, `no_key_for_provider`, `denied`,
  `invalid_or_expired_token`, `kill_switch_active`, `monthly_cap_reached`,
  `reveal_not_permitted`.

A full, runnable example lives in [`examples/integration-example`](examples/integration-example).

> **Web apps:** the same `chrome.runtime.sendMessage(id, msg, cb)` call works
> from any HTTPS page (GhostKey allows external connections from `https://*/*`).
> The user still approves every new requester via the permission prompt.

## Security model

**Threat model.** GhostKey protects your API keys from (a) other extensions and
web apps that would otherwise receive them in the clear, (b) casual disk
inspection of extension storage, and (c) accidental pasting of keys into web
pages. It is **not** designed to defend against a fully compromised OS/browser
or a keylogger capturing your master passphrase.

**Cryptography.**
- Keys are stored only as **AES-256-GCM** ciphertext in `chrome.storage.local`.
- The encryption key is derived from your master passphrase with
  **PBKDF2-HMAC-SHA-256, 600,000 iterations** and a random per-vault salt.
- The derived key is **non-extractable** by default and lives only in the
  service worker's memory while unlocked.
- A random 96-bit IV is generated for **every** encryption (never reused).
- GCM's authentication tag detects any tampering with the vault or a wrong passphrase.

**Access control.**
- No requester ever gets a key without an explicit user grant.
- Default grants are **proxy-only** — the raw key is never returned.
- Sessions are **short-lived** and revocable; "Always allow" is remembered per
  requester+provider and can be removed anytime.
- A **kill-switch** and **monthly cap** can refuse all access instantly.

**Privacy.**
- No servers, no accounts, no analytics, no network calls except the provider
  requests **you** authorize (proxied on your behalf).
- The audit log and usage stats never contain key material and never leave your device.

**What we deliberately avoid.**
- Your passphrase is never stored, transmitted, or recoverable.
- "Keep unlocked this session" (opt-in, off by default) caches the key in
  `chrome.storage.session` (in-memory, wiped on browser close) — a documented
  convenience/security trade-off you control.

**Audit it yourself.** All crypto is in [`src/lib/crypto.js`](src/lib/crypto.js);
vault logic in [`src/lib/vault.js`](src/lib/vault.js); the request protocol in
[`src/lib/messaging.js`](src/lib/messaging.js). No minification, no build step.

## Permissions rationale

| Permission | Why |
| --- | --- |
| `storage` | Store the **encrypted** vault, settings, audit log, usage. |
| `sidePanel` | The main vault UI. |
| `alarms` | Auto-lock timer. |
| `externally_connectable` | Let other extensions/apps request access via the protocol. |
| content script | The clipboard paste-guard reads only the text of a paste event locally — **no** `clipboardRead` permission needed. |
| host access | Not requested broadly; proxying uses `fetch` to the provider you configured. |

GhostKey deliberately requests **no broad host permissions** and **no**
`clipboardRead`/`notifications`. The proxy uses `fetch` under the extension's
own origin to the provider endpoint you configured.

## Project structure

```
ghostkey/
├── manifest.json
├── LICENSE                     # MIT
├── README.md
├── PACKAGING.md                # build a Web Store zip
├── icons/                      # 16/32/48/128 PNG + source SVG
├── src/
│   ├── background/service-worker.js   # routing, auto-lock, external protocol
│   ├── lib/
│   │   ├── crypto.js           # AES-GCM + PBKDF2 (audit me first)
│   │   ├── vault.js            # unlock/lock, key CRUD, export/import
│   │   ├── storage.js          # chrome.storage wrappers
│   │   ├── constants.js        # providers, pricing, message types
│   │   ├── messaging.js        # external protocol + permission flow
│   │   ├── sessions.js         # short-lived session tokens
│   │   ├── audit.js            # append-only audit log
│   │   ├── usage.js            # usage + cost + caps
│   │   └── ghostkey-client.js  # public SDK (web-accessible)
│   ├── sidepanel/              # full UI
│   ├── popup/                  # quick access
│   ├── permission/             # consent window
│   ├── onboarding/             # first-run
│   ├── content/clipboard-guard.js
│   └── ui/                     # theme + shared helpers
└── examples/integration-example/   # runnable demo integrator
```

## Contributing

Issues and PRs welcome. Because there's no build step, you can edit files and
hit **Reload** on `chrome://extensions`. Please keep the crypto path dependency-free
and easy to audit. Adding a provider is usually just an entry in
`src/lib/constants.js`.

## License

[MIT](LICENSE) © GhostKey contributors.
