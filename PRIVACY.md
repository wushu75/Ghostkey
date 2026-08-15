# GhostKey Privacy Policy

_Last updated: 2026_

GhostKey is designed so there is almost nothing to say here — and that's the point.

## The short version

GhostKey does not collect, transmit, or sell any of your data. It has no
servers, no analytics, no accounts, and no telemetry. Everything stays on your
device.

## What GhostKey stores, and where

GhostKey stores the following **locally on your device**, using the browser's
extension storage:

- Your AI API keys, **encrypted** with AES-256-GCM using a key derived from your
  master passphrase (PBKDF2-HMAC-SHA-256, 600,000 iterations). Keys are never
  stored in plaintext.
- Your settings (theme, auto-lock timeout, spend caps, etc.).
- An on-device audit log and usage/cost statistics. These never contain key
  material.
- Records of which applications you have granted access to.

None of this is ever sent to the developer or any third party. Your master
passphrase is never stored, transmitted, or recoverable.

## Network requests

The only network requests GhostKey makes are the AI-provider API calls **you
authorize** — for example, when you approve an app's request and GhostKey
proxies it to OpenAI, Anthropic, Moonshot, or another provider you configured.
Those requests go directly from your browser to that provider under your own
API key. GhostKey does not route them through any intermediary.

## Data sharing

GhostKey does not share data with anyone. Access you grant to other
applications is delegated locally by you, is revocable at any time, and — in the
default proxy mode — never exposes your raw key to those applications.

## Your control

You can lock the vault, revoke any or all granted sessions, rotate or delete
keys, export an encrypted backup, or remove the extension entirely at any time.
Uninstalling the extension removes its local storage.

## Contact

Questions or issues: https://github.com/wushu75/Ghostkey/issues _(replace with
your real repository)._
