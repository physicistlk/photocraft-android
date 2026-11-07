## 0.21.0 (2026-11-07)

- Re-port to upstream v0.21.0 (commit efb7ebd0c).
- Patch applied cleanly.
- APK: photocraft-0.21.0-arm64-v8a-20261107.apk (sha256 fe6f3f3463d2f53b5b0bac4076f7b415941393490e655b95747cbd4053ba5c7d)

# Changelog

## 2026-11-04 — Re-pinned to upstream 0.20.1

| | |
|---|---|
| Upstream pin | `327c4cc82e737a3e0af7c147a477126d77525495` (upstream `0.20.1`) — was `2df9f8077` (`0.19.0`) |
| APK | `photocraft-0.20.1-arm64-v8a-20261104.apk` — sha256 `ebd39e91073d7cd6a3c449c057d85434b53c42d6c94fd2de48ff2032b246743e` |
| Package | `ai.storyteller.photocraft` — versionName `0.20.1`, minSdk 26, targetSdk 35 |
| ABI | `arm64-v8a` only |

### Changed
- **The upstream pin moved from 0.19.0 to 0.20.1.** The existing port patch applied cleanly (hunk offsets shifted only); no semantic re-derivation was needed. `cargo check -p photocraft-android --target aarch64-linux-android` passes with no changes to the shell crate.
- **Patch regenerated** against the 0.20.1 base and verified with `git apply --check` against a pristine `git archive v0.20.1` tree.

## 2026-10-30 — The system clipboard (text)

| | |
|---|---|
| APK | `photocraft-0.17.0-arm64-v8a-20261030.apk` — sha256 in `SHA256SUMS.txt` beside the build |
| Package | `ai.storyteller.photocraft` — versionName `0.17.0`, minSdk 26, targetSdk 35 |
| ABI | `arm64-v8a` only |

### Changed
- **The system clipboard works for text (`src/clipboard.rs`).** Ctrl+C/X/V now produce
  `Event::Copy`/`Cut`/`Paste` (the Android backend never reports modifiers, so `egui-winit` never
  did), Paste reads `ClipboardManager` through JNI, and `OutputCommand::CopyText` is written back
  with `ClipData.newPlainText`. Ctrl+Shift (Copy Merged) and Ctrl+Alt (Paste Special) still reach
  the app. Plain text only: the image clipboard fields on `Services` stay unset.

## Unreleased — re-pinned to upstream 0.17.0, Android 16 (16 KB page size)

| | |
|---|---|
| Upstream pin | `b8c3f3aa8e2918032cadc0e042664414a03be333` (upstream `0.17.0`) — was `7a6058e` (`0.7.0`) |
| Package | `ai.storyteller.photocraft` — versionName `0.17.0`, minSdk 26, targetSdk 35 |
| ABI | `arm64-v8a` only |
| ELF page alignment | `0x4000` (16 KB) — Android 16 compatible |

### Changed
- **The upstream pin moved from 0.7.0 to 0.17.0.** Both patched upstream files were re-derived
  against the new tree; the old hunks did not apply and the patch was rebuilt from the new base.
- **Android 16 page-size compatibility.** The native library is linked with
  `-Wl,-z,max-page-size=16384`, so every `LOAD` segment is 16 KB aligned. Android 16 shows a
  compatibility warning for a 4 KB-aligned app on a 16 KB-page device; this build is not one.

### Fixed
- **`Services::import` takes the whole `ImportOptions`** upstream now carries a second option (the
  raw demosaic method), so the shell passes the options through instead of a bare SVG depth.
- **The in-app picker answers a folder request.** Upstream added
  `FileDialogRequest::Folder { initial_dir }`; the picker browses folders only and answers the
  folder the user is in.


## v0.1.1 — 2026-10-13 (scrolling, modifiers, fonts, picker wording)

| | |
|---|---|
| APK | `photocraft-0.7.0-arm64-v8a-20261013.apk` (25.2 MB) — supersedes the v0.1.0 asset, same signing key |
| sha256 | `4cfcfc2a87399f23c3af67c4eb16907e58550f648dc09c9a716c15377a926e27` |
| Built from | upstream `storytold/photocraft` @ `7a6058e6b4a5dbb725e75e97f7ee019d2bfb7f0f` + this repository at the commit that released it |
| Package | `ai.storyteller.photocraft` — versionName `0.7.0`, versionCode `16779008`, minSdk 26, targetSdk 35 |
| ABI | `arm64-v8a` only |

### Changed
- **A save no longer blocks the event loop.** Long commands and the document write now run through the
  app's background-job path (`background_jobs = true` plus the worker-thread `save_file` service that
  path needs), instead of inline on the event loop — the ~5-second stall that had Android offering
  *Force close* mid-save.
- **Picker wording corrected.** An empty folder in shared storage no longer tells a user who *has*
  granted *All files access* to grant it; the note is conditional and leaves the decision to the user.
  The USB tip now depends on the folder (`Android/data` cannot be reached over a cable; `Android/media`
  can).

### Notes
The build was made from exactly the source in this repository (upstream pin + the patch + the port
crate) and signed with the same local test key as v0.1.0, so it installs over it.


## v0.1.0 — first published port (PhotoCraft 0.7.0)

| | |
|---|---|
| APK | `photocraft-0.7.0-arm64-v8a-fonts-keys.apk` (25.2 MB) |
| sha256 | `314cf5dbc062c3bcef8233f55d89a2b3a627c881760c0d4eb6a12033de373cc9` |
| Built from | upstream `storytold/photocraft` @ `7a6058e6b4a5dbb725e75e97f7ee019d2bfb7f0f` + `patches/0001-photocraft-android-port.patch` + `apps/photocraft-android/` |
| Package | `ai.storyteller.photocraft` — versionName `0.7.0`, minSdk 26, targetSdk 35 |
| ABI | `arm64-v8a` only |
| Signature | throwaway local test key, not an official key |

### In this release
- Android shell crate: NativeActivity entry point, preferences under the app's own data directory,
  file logger, single-window lifecycle.
- In-app file picker drawn with egui, with readable-root probing, "Show all files", hidden-file
  count, and a locked root that names the permission it needs.
- Edge-to-edge fix for `targetSdkVersion 35` (`res/values/styles.xml`, variant B immersive).
- **File ▸ Exit** now really ends the process (it previously appeared to hang on a stale close).
- **Ctrl / Shift / Alt** work: the platform reports no modifiers, so they are tracked from key events
  and re-injected (5 host unit tests, all passing).
- **System fonts** (`/system/fonts` and friends) plus a USB-visible *Fonts* folder a user can drop a
  `.ttf` into.
- HEIC/HEIF opening enabled by default (as the official builds compile it).
- App icon (adaptive) and launcher label; `MANAGE_EXTERNAL_STORAGE` + media permissions declared.


### Known issues
See *Known gaps* in the [README](README.md) and [`docs/PORT-NOTES.md`](docs/PORT-NOTES.md).
