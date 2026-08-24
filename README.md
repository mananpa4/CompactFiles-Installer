# CompactFiles Installer

Distribution and auto-update channel for [CompactFiles](https://github.com/mananpa/CompactFiles) and CompactFiles Legacy. This repo does **not** contain source code — it only hosts installer binaries (as GitHub Release assets) and the [`version.json`](version.json) manifest that both apps poll to check for updates.

## How the auto-updater works

Both apps periodically fetch:

```
https://raw.githubusercontent.com/mananpa4/CompactFiles-Installer/main/version.json
```

and compare the version there against their own running version. If a newer one is found, the app downloads the matching installer directly from this repo's **latest GitHub Release** using the stable shorthand URL:

```
https://github.com/mananpa4/CompactFiles-Installer/releases/latest/download/<asset-filename>
```

That URL always resolves to whatever is attached to the most recent release, regardless of its tag name — so a new release just needs the matching filenames re-uploaded, no app-side changes required.

## `version.json` schema

Two independent products are tracked, each with its own version line (they don't ship together):

```json
{
  "compactFiles": {
    "latest": { "major": 4, "minor": 0, "patch": 0, "preRelease": "beta", "preReleaseMinor": 6 },
    "latestNonPreRelease": { "major": 4, "minor": 0, "patch": 0, "preRelease": "", "preReleaseMinor": 0 },
    "assets": {
      "x64": "CompactFiles_<version>_Win10-11_x64_Setup.exe",
      "x86": "CompactFiles_<version>_Win10-11_x86_Setup.exe",
      "arm64": "CompactFiles_<version>_Win10-11_arm64_Setup.exe"
    }
  },
  "compactFilesLegacy": {
    "latest": { "major": 1, "minor": 0, "patch": 0, "preRelease": "", "preReleaseMinor": 0 },
    "latestNonPreRelease": { "major": 1, "minor": 0, "patch": 0, "preRelease": "", "preReleaseMinor": 0 },
    "assets": {
      "anycpu": "CompactFilesLegacy_<version>_Win7-8.1_Setup.exe"
    }
  }
}
```

- `latest` — newest version, including pre-releases. Used when the user opted in to pre-release updates.
- `latestNonPreRelease` — newest stable version. Used by default.
- `assets` — maps each build target to the exact filename attached to the GitHub Release. The app picks the key matching its own architecture (`x64` / `x86` / `arm64` for CompactFiles; `anycpu` for Legacy, since it's a single binary that runs on both x86 and x64 Windows 7/8/8.1).
- A version only counts as "available" if it compares greater than the running app's version (major → minor → patch, then pre-release tag).

## Asset naming convention

`<Product>_<version>_<SupportedWindows>_<arch>_Setup.exe`

| Product | Supported Windows | Architectures | Example |
|---|---|---|---|
| `CompactFiles` | Win10-11 (`MinVersion=10.0.17763`) | `x64`, `x86`, `arm64` | `CompactFiles_4.0.0-beta.6_Win10-11_x64_Setup.exe` |
| `CompactFilesLegacy` | Win7-8.1 (via `wofadk.sys`) | single AnyCPU build, runs on both 32/64-bit hardware | `CompactFilesLegacy_1.0.0_Win7-8.1_Setup.exe` |

The `<SupportedWindows>` segment exists specifically so a user can tell at a glance which installer targets which Windows version, without having to guess from the architecture alone.

## Cutting a release

1. Build the installers from the main [CompactFiles](https://github.com/mananpa/CompactFiles) repo (`build_x64_arm64.ps1` for the modern app, `CompactFiles.Legacy.iss` for Legacy).
2. Rename each output to match the convention above.
3. Create (or update) a GitHub Release in this repo and attach the renamed installers as assets. Any tag works — the update URLs always pull from *whatever release is currently "latest"*, not from a specific tag.
4. Update `version.json` at the repo root with the new version number(s) and asset filenames, and push to `main`.
5. Only update the version number for the product that actually changed — CompactFiles and CompactFiles Legacy version independently.

Until both `version.json` and the matching release assets are in place, do not bump the version numbers — the updater has no fallback if the asset it expects is missing from the release.
