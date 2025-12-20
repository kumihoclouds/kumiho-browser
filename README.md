# kumiho-browser (Distribution Repo)

This repository is intended to be the **public distribution feed** for the Kumiho Browser desktop app.

The app can be built to check this repo for updates (without any embedded GitHub tokens) by setting build-time Dart defines:

- `UPDATE_GITHUB_OWNER`
- `UPDATE_GITHUB_REPO`

Example:

```bash
flutter build windows --release \
  --dart-define=UPDATE_GITHUB_OWNER=kumihoclouds \
  --dart-define=UPDATE_GITHUB_REPO=kumiho-browser
```

## In-app updates: how it works

The desktop app’s updater uses the **GitHub Releases API**:

- Latest release endpoint:
  - `https://api.github.com/repos/<owner>/<repo>/releases/latest`

From that response it reads:

- `tag_name` (expected format: `browser-vX.Y.Z`)
- `assets[]` (download URLs)

The updater compares the running app version (`X.Y.Z`) to the latest release version. If newer, it selects platform-specific assets:

- **Windows**: prefers an `.exe` installer asset containing `KumihoBrowserSetup` in the filename.
  - When the user clicks **Download & Install**, the app downloads the installer to a temp folder, launches it, then exits.
- **Linux**: looks for `.deb` and `.rpm` assets and offers download buttons.
- **macOS**: currently treated as “manual update” (the app links users to the release page).

Notes:

- This design intentionally avoids private repos and avoids shipping secrets inside the app.
- GitHub API calls are unauthenticated, so very high traffic could hit rate limits. For most early-stage usage this is fine.

## Publishing a new version

To publish an update that the app can see:

1. Create a **GitHub Release in this repo**.
2. Use a tag name like:

   - `browser-v1.2.3`

3. Upload the release assets you want users to install.

### Recommended assets / naming

These names are what the updater will pick most reliably:

- Windows installer (required for in-app install on Windows)
  - `KumihoBrowserSetup-1.2.3.exe` (any `KumihoBrowserSetup-*.exe` is fine)

- Linux packages
  - `kumiho-browser_1.2.3_amd64.deb`
  - `kumiho-browser-1.2.3-1.x86_64.rpm`

- macOS artifacts (manual install)
  - `.dmg` preferred, otherwise a macOS `.zip`

## Sparkle (macOS auto-update)

Some build pipelines also generate Sparkle-related files (e.g. `appcast.xml` and a `.zip.signature`).

At the moment, the in-app update flow described above is based on **GitHub Releases**, and macOS is **manual**.

If/when you enable Sparkle for macOS auto-update, you’ll typically:

- publish a signed update `.zip`
- publish `appcast.xml`
- host them somewhere stable (GitHub Pages, a CDN, etc.)

## Troubleshooting

- If the app never finds updates:
  - Ensure this repo has a GitHub Release with tag `browser-vX.Y.Z`.
  - Ensure the app build used `UPDATE_GITHUB_OWNER/UPDATE_GITHUB_REPO` pointing here.
- If Windows shows “update available” but no installer button:
  - Make sure the release contains an `.exe` asset (preferably named `KumihoBrowserSetup-*.exe`).
