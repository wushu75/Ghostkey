# Packaging GhostKey for the Chrome Web Store

GhostKey has **no build step** — the source is exactly what ships. Packaging is
just zipping the right files.

## 1. What to include / exclude

Ship everything the extension needs at runtime and nothing else:

**Include**
- `manifest.json`
- `icons/` (the four PNGs; the SVG is optional)
- `src/` (all of it — background, lib, UIs, content)

**Exclude** (not needed at runtime; keep the zip clean)
- `examples/`
- `docs/`
- `README.md`, `PACKAGING.md`, `LICENSE` *(optional to include; harmless)*
- `.git/`, `.github/`, `.DS_Store`, `node_modules/`, any `*.map`

## 2. Bump the version

Edit `"version"` in `manifest.json` (e.g. `1.0.0` → `1.0.1`). The Web Store
rejects re-uploads that don't increase the version.

## 3. Build the zip

From the project root:

```bash
# macOS / Linux
zip -r ghostkey.zip manifest.json icons src \
  -x "*/.DS_Store" "*/node_modules/*" "*.map"
```

```powershell
# Windows PowerShell
Compress-Archive -Path manifest.json, icons, src -DestinationPath ghostkey.zip -Force
```

Or use the helper script:

```bash
bash scripts/build-zip.sh      # produces dist/ghostkey-<version>.zip
```

## 4. Verify the zip loads

Before submitting, unzip into a temp folder and **Load unpacked** it in
`chrome://extensions`. Confirm: onboarding opens, you can create a vault, add a
key, and the popup + side panel work. This catches path/casing mistakes that
only appear in a fresh unpack.

## 5. Submit

1. Go to the [Chrome Web Store Developer Dashboard](https://chrome.google.com/webstore/devconsole).
2. **Add new item** → upload `ghostkey.zip`.
3. Fill in listing details. Suggested copy:
   - **Summary:** "Local-first, encrypted BYOK wallet for AI API keys. Your keys never leave your device."
   - **Category:** Productivity / Developer Tools.
4. **Privacy practices:** declare **no data collected**. GhostKey has no servers,
   no analytics, and makes no network calls except the provider requests the
   user authorizes. Justify each permission using the table in the README.
5. Provide screenshots (1280×800 or 640×400) of the vault, permission prompt,
   and dashboard.
6. Submit for review.

## 6. Store assets checklist

- [ ] 128×128 store icon (`icons/icon128.png`)
- [ ] At least one 1280×800 screenshot
- [ ] Short + detailed description
- [ ] Privacy policy URL (a simple page stating "no data leaves the device")
- [ ] Single purpose statement: "Securely store and delegate access to AI API keys."

## Notes for Edge / other Chromium stores

The same zip works for the **Microsoft Edge Add-ons** store and most Chromium
browsers. Edge's dashboard is separate but the package format is identical.
