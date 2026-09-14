# Callcap Desktop Recorder — Self-Hosted

Electron app that records both sides of a call (microphone + system audio) and
uploads to the self-hosted Callcap instance at **callcap.quietly.ch**.

This is an independent fork of `Joel-bre/callcap-desktop` (the Lovable-hosted
recorder), pointed at the self-hosted backend, with its own app identity so the
two never conflict on one machine:

| | Lovable recorder | This one |
| --- | --- | --- |
| Backend | `callcap.lovable.app` | `callcap.quietly.ch` |
| Bundle ID | `app.callcap.recorder` | `app.callcap.recorder.selfhost` |
| URL scheme | `callcap://` | `callcap-sh://` |
| Releases | `Joel-bre/callcap-desktop` | `Joel-bre/callcap-desktop-selfhost` (this repo) |

Both can be installed side by side. Neither auto-updates the other.

## Cutting a release

```bash
git tag v1.0.1
git push --tags
```

GitHub Actions builds and publishes:
- `Callcap-SelfHosted.dmg` + `.zip` (macOS, universal — Intel + Apple Silicon)
- `Callcap-Setup.exe` (Windows)

See `SIGNING.md` for the macOS signing/notarization secrets — without them the
macOS job is skipped (Windows still ships). Windows ships unsigned regardless
(users click through one SmartScreen warning per version).

The web app's `/download` page points at `releases/latest/download/...`, so it
always serves the newest build. The recorder self-updates via `electron-updater`
against this repo's releases.

## Local dev

```bash
npm install
npm start
```

## How pairing works

1. User opens `callcap.quietly.ch/pair` → clicks "Pair this device".
2. Browser opens `callcap-sh://pair?token=<short-lived-token>` (falls back to a
   copy-pasteable token if the OS doesn't hand off the link).
3. This app POSTs the token to `/api/public/recorder/pair` → receives a
   long-lived `device_token`, stored encrypted via the OS keychain.
4. Recordings upload as two-channel audio — mic on channel 0, system audio on
   channel 1 — tagged `channel_layout=mic_remote` so the server transcribes
   each side separately.
