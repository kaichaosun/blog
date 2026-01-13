---
date: 2026-01-13
title: Tauri App Signing & Release Guide
tags: ["Rust", "Build in Public"]
categories: ["Rust"]
---

This guide covers how to sign and release your Kanban app for macOS, Windows, and Linux.

## Quick Start

1. Set up signing certificates (see platform-specific sections below)
2. Add secrets to GitHub repository
3. Push a version tag: `git tag v1.0.0 && git push origin v1.0.0`
4. The GitHub Action will build, sign, and create a draft release

---

## macOS Code Signing & Notarization

### Prerequisites
- Apple Developer Account ($99/year)
- Mac computer with Xcode installed

### Step 1: Create a Developer ID Certificate

1. On your Mac, open **Keychain Access**
2. Go to **Keychain Access > Certificate Assistant > Request a Certificate from a Certificate Authority**
3. Enter your email, select "Saved to disk", and save the `.certSigningRequest` file

4. Go to [Apple Developer Certificates](https://developer.apple.com/account/resources/certificates/list)
5. Click **+** to create a new certificate
6. Select **Developer ID Application** (for distribution outside App Store)
7. Upload your `.certSigningRequest` file
8. Download the certificate and double-click to install it in Keychain

### Step 2: Export Certificate for CI/CD

1. Open **Keychain Access**
2. Find your certificate under **My Certificates**
3. Expand it, right-click the private key, select **Export**
4. Save as `.p12` file with a strong password
5. Convert to base64:
   ```bash
   base64 -i Certificates.p12 -o certificate-base64.txt
   ```

### Step 3: Get Your Signing Identity

Run this command to find your signing identity:
```bash
security find-identity -v -p codesigning
```

Look for something like:
```
"Developer ID Application: Your Name (TEAMID123)"
```

### Step 4: Create App-Specific Password (for notarization)

1. Go to [appleid.apple.com](https://appleid.apple.com)
2. Sign in > Security > App-Specific Passwords
3. Generate a new password for "Tauri Notarization"

### Step 5: Find Your Team ID

1. Go to [Apple Developer Membership](https://developer.apple.com/account#MembershipDetailsCard)
2. Your Team ID is shown there

### Step 6: Add GitHub Secrets

Go to your repo > Settings > Secrets and variables > Actions, add:

| Secret | Value |
|--------|-------|
| `APPLE_CERTIFICATE` | Contents of `certificate-base64.txt` |
| `APPLE_CERTIFICATE_PASSWORD` | Password used when exporting .p12 |
| `APPLE_SIGNING_IDENTITY` | e.g., `Developer ID Application: Your Name (TEAMID123)` |
| `APPLE_ID` | Your Apple ID email |
| `APPLE_PASSWORD` | App-specific password from Step 4 |
| `APPLE_TEAM_ID` | Your Team ID from Step 5 |

### Local Build with Signing

```bash
export APPLE_SIGNING_IDENTITY="Developer ID Application: Your Name (TEAMID123)"
export APPLE_ID="your@email.com"
export APPLE_PASSWORD="app-specific-password"
export APPLE_TEAM_ID="TEAMID123"

pnpm tauri build
```

---

## Windows Code Signing (Optional)

Windows signing prevents SmartScreen warnings but requires purchasing a code signing certificate ($200-400+/year).

### Option 1: OV Certificate (cheaper, still shows initial warning)

1. Purchase from DigiCert, Sectigo, or similar
2. Convert to PFX format:
   ```bash
   openssl pkcs12 -export -in cert.cer -inkey private-key.key -out certificate.pfx
   ```
3. Add to GitHub Secrets:
   - `WINDOWS_CERTIFICATE`: base64 of .pfx file
   - `WINDOWS_CERTIFICATE_PASSWORD`: export password

### Option 2: Skip Signing

Windows apps work without signing, but users will see a SmartScreen warning on first download. They can click "More info" > "Run anyway".

---

## Linux

Linux builds don't require code signing. The workflow produces:
- `.deb` (Debian/Ubuntu)
- `.rpm` (Fedora/RHEL)
- `.AppImage` (Universal)

---

## Tauri Configuration

Update `src-tauri/tauri.conf.json`:

```json
{
  "productName": "Kanban",
  "version": "1.0.0",
  "identifier": "com.yourcompany.kanban",
  "bundle": {
    "active": true,
    "category": "Productivity",
    "copyright": "© 2025 Your Company",
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/128x128@2x.png",
      "icons/icon.icns",
      "icons/icon.ico"
    ],
    "macOS": {
      "minimumSystemVersion": "10.13",
      "hardenedRuntime": true
    },
    "windows": {
      "certificateThumbprint": null,
      "digestAlgorithm": "sha256",
      "timestampUrl": "http://timestamp.digicert.com"
    }
  }
}
```

---

## Release Process

### 1. Update Version

Update version in these files:
- `package.json`
- `src-tauri/Cargo.toml`
- `src-tauri/tauri.conf.json`

### 2. Commit and Tag

```bash
git add .
git commit -m "Release v1.0.0"
git tag v1.0.0
git push origin main --tags
```

### 3. Monitor Build

1. Go to **Actions** tab in GitHub
2. Watch the "Release" workflow
3. Once complete, go to **Releases**
4. Edit the draft release, add release notes
5. Click **Publish release**

---

## Troubleshooting

### macOS: "Team is not yet configured for notarization"

Contact Apple Developer Support. New accounts sometimes need manual activation for notarization.

### macOS: Notarization takes too long

Check status:
```bash
xcrun notarytool history --apple-id YOUR_APPLE_ID --password YOUR_APP_PASSWORD --team-id YOUR_TEAM_ID
```

### Windows: SmartScreen warning

This is normal for new/unsigned apps. Options:
1. Purchase an EV certificate (expensive, no warning)
2. Build reputation over time with OV certificate
3. Submit to Microsoft for manual review

### Build fails on GitHub Actions

Check that all secrets are properly set and the certificate hasn't expired.

---

## Alternative: Ad-hoc Signing (No Apple Developer Account)

For testing or personal use on Apple Silicon Macs:

```bash
export APPLE_SIGNING_IDENTITY="-"
pnpm tauri build
```

Users will need to right-click > Open the first time.
