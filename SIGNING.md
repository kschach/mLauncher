# Signing Setup

One-time setup to get CI builds signing and releasing correctly.

## 1. Generate a keystore (run locally, not in CI)

```bash
keytool -genkey -v \
  -keystore mLauncher.jks \
  -alias mlauncher \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000
```

You'll be prompted for:
- A keystore password
- A key password (can be the same as the keystore password)
- Identity info (name, org, country — can be left blank for personal projects)

**Keep `mLauncher.jks` somewhere safe and never commit it to the repo.**

## 2. Base64-encode the keystore

```bash
# macOS
base64 -i mLauncher.jks | tr -d '\n'

# Linux
base64 -w 0 mLauncher.jks
```

Copy the output — you'll need it in the next step.

## 3. Add GitHub Secrets

Go to `kschach/mLauncher` → **Settings → Secrets and variables → Actions → New repository secret**

| Secret name | Value |
|---|---|
| `SIGNINGKEY_BASE64` | the base64 output from step 2 |
| `KEY_ALIAS` | `mlauncher` (or whatever alias you chose) |
| `KEY_STORE_PASSWORD` | the keystore password from step 1 |
| `KEY_PASSWORD` | the key password from step 1 |
| `GIT_BOT_TOKEN` | a GitHub personal access token (classic, `repo` scope) |

## 4. Verify

Push any commit to a non-main branch. The `Android Feature Branch CI` workflow
should build successfully. A push to `main` will trigger `Android Main Branch CI`.

## Notes

- One keystore per app is recommended for published apps. If you have other Android
  projects, generate a separate `.jks` for each.
- If you use Google Play, the key you sign with is permanent — Play ties the app's
  update chain to it. Back up `mLauncher.jks` securely (password manager, cloud
  backup, etc.).
