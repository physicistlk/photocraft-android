# PhotoCraft for Android (arm64-v8a)

An unofficial Android port of **[PhotoCraft](https://github.com/storytold/photocraft)** — the real engine
and egui interface, packaged as a NativeActivity APK for sideloading on arm64 devices.

| | |
|---|---|
| Upstream project | [`storytold/photocraft`](https://github.com/storytold/photocraft) — Apache-2.0 OR MIT |
| Pinned commit | `45066c482` (upstream version `0.23.1`) |
| Android package | `ai.storyteller.photocraft` |
| Published APK | `photocraft-0.23.1-arm64-v8a-20261112.apk` — 39 MB, arm64-v8a, signed (2026-11-12) |
| APK sha256 | `2c4eff576f9073970d6198ff9e3044047b1e1be9e5c90136d22f55b9f2564825` |
| Upstream files changed | 4 files — see [`patches/`](patches/) |
| New port crate | `apps/photocraft-android/` — 2045 lines of Rust plus resources |

## What this repository is

This is **not a fork** of PhotoCraft. It holds only the Android layer, so it stays small, readable,
and obviously separable from upstream:

```
apps/photocraft-android/     the new shell crate — copy it into your checkout of upstream
patches/                the diff against upstream — apply it after checking out the pin
docs/PORT-NOTES.md      what was changed and why, fix by fix, with how each one was checked
scripts/                cross-compile environment, build and verify scripts (from shared-android-port)
CHANGELOG.md            what is in the published APK
```

The Android layer is not a reimplementation. It is a shell crate that fills the same `Services`
struct the desktop and web shells fill, so the real engine and the real egui UI run unchanged.

## Build it

Prerequisites: Rust (rustup) with the `aarch64-linux-android` target, Android SDK + NDK r27, a
JDK 21, and `cargo install cargo-apk`.

```bash
git clone https://github.com/storytold/photocraft.git
cd photocraft
git fetch --depth 1 origin 45066c482
git checkout FETCH_HEAD                        # the exact commit this port targets
git apply -3 /path/to/0001-photocraft-android-port.patch
cp -r /path/to/photocraft-android apps/             # the shell crate from this repository

# any keystore of your own signs an installable APK; a throwaway test key is fine for sideloading
keytool -genkeypair -keystore ks.jks -alias photocraft -keyalg RSA -keysize 2048 -validity 10000 \
        -storepass android -keypass android -dname "CN=Local Test, O=Local, C=US"
export CARGO_APK_RELEASE_KEYSTORE="$PWD/ks.jks" CARGO_APK_RELEASE_KEYSTORE_PASSWORD=android
cargo apk build --release -p photocraft-android
```

`scripts/android-env.sh` is the cross-compile environment this port was built with (the NDK linker
and the C compiler the native dependencies need), `scripts/build-apk.sh` wraps the build, and
`scripts/verify-apk.sh` checks the artifact: `aapt2 dump badging`, `apksigner verify`, and `llvm-nm`
on the `.so` inside the APK.

## Install

```bash
adb install -r photocraft-0.7.0-arm64-v8a-fonts-keys.apk
```
or copy the APK to the device and open it (installing from unknown sources must be allowed).

On first run, grant **Settings ▸ Apps ▸ PhotoCraft ▸ Special access ▸ All files access** if you want
the in-app picker to reach files outside the app's own folders. Without it, Android 10+ refuses a
`read_dir` of shared storage by path, and the picker can only offer the app's own directory.

## Verification status

Everything below was checked on the machine this port was published from, in October 2026.
A tick means the check was *run*, not inferred; the wording says how far it goes.

| Claim | How it was checked | Result |
|---|---|---|
| `patches/…patch` applies to a clean checkout of the pin | fresh `git init` + `git fetch --depth 1 origin 45066c482` + `git apply --check` | ✅ applies cleanly |
| The assembled tree equals the tree the APK was built from | file-by-file SHA-256 (line endings normalised) over every patched file and every file of the port crate | ✅ 17 files identical, none differing |
| The port crate compiles for Android | `cargo check -p photocraft-android --target aarch64-linux-android`, run in a tree assembled from scratch (fresh `git fetch` of the pin, patch applied, port crate copied in) | ✅ exit 0 (host `cargo check -p photocraft` stays green after the shared-crate patch) |
| Modifier-key synthesis (Ctrl / Shift / Alt) | `cargo test -p photocraft-android` (host, pure logic) | ✅ 14/14 pass, including the cross-frame Shift case that *is* the bug |
| The port crate's own tests | `cargo test -p photocraft-android` in the freshly assembled tree | ✅ 14/14 pass |
| APK is signed and is the right artifact | `apksigner verify --verbose`, `aapt2 dump badging`, sha256 | ✅ v2/v3 signatures true, `native-code: 'arm64-v8a'`, package `ai.storyteller.photocraft` |
| Behaviour on real hardware | hand-testing on the porter's arm64 tablet | ⚠️ the fixes below were found and driven by that testing; each published binary is not individually re-verified here |

**Before trusting the APK:** the source side is verified as listed above; runtime behaviour only as far
as [`docs/PORT-NOTES.md`](docs/PORT-NOTES.md) says per fix, plus the tablet testing that produced those
fixes. There is no automated device test in this repository.

## Known gaps

- **Runtime verification is hand-testing only.** No automated device test exists here; see
  *Verification status* above for exactly what was and was not checked.
- **No second window.** PhotoCraft has no *New Window* command on Android; one document at a time per
  Activity.
- **The system clipboard is text only.** Copy/Paste crossed the app boundary for text;
  pictures and HTML on the clipboard are still out of reach (`ClipData` carries them, but this
  port does not negotiate those formats yet).
- **All files access is a manual grant.** Until it is given, the picker offers only the app's own
  directory — by design, and it says so on the locked root.
- **Android 16 ignores the status-bar opt-out.** Variant A of the theme is Android 15 only; the
  proper fix is a JNI `setOnApplyWindowInsetsListener` plus an egui top spacer, which is not
  implemented here.
- **A system font is only as good as `/system/fonts`.** If a device keeps a family out of there, the
  app cannot see it; the *Fonts* folder under `Android/media/…` is the way round that.


## The rest of this set

Every port in this set is built from the same patched upstream tree and published by
[@physicistlk](https://github.com/physicistlk). Each link goes to that port's latest
release, which carries the signed `arm64-v8a` APK:

- [cadcraft-android](https://github.com/physicistlk/cadcraft-android/releases/latest) — CadCraft 0.20.1
- [deckcraft-android](https://github.com/physicistlk/deckcraft-android/releases/latest) — DeckCraft 0.18.1
- [gridcraft-android](https://github.com/physicistlk/gridcraft-android/releases/latest) — GridCraft 0.19.1
- [lightcraft-android](https://github.com/physicistlk/lightcraft-android/releases/latest) — LightCraft 0.21.1
- [pdfcraft-android](https://github.com/physicistlk/pdfcraft-android/releases/latest) — PdfCraft 0.21.1
- [vectorcraft-android](https://github.com/physicistlk/vectorcraft-android/releases/latest) — VectorCraft 0.24.1
- [wordcraft-android](https://github.com/physicistlk/wordcraft-android/releases/latest) — WordCraft 0.18.1

## Credits and licence

PhotoCraft is © its authors at [`storytold/photocraft`](https://github.com/storytold/photocraft) and is
dual-licensed Apache-2.0 / MIT; both licence texts are kept here. The Android layer in this
repository is an unofficial modification by [@kanaaka](https://github.com/kanaaka), carrying the
same licence. Not affiliated with or endorsed by the upstream project. See
[`ATTRIBUTION.md`](ATTRIBUTION.md).


## Android 16 build (physicistlk fork)

This fork carries the Android 16 port (arm64-v8a, 16 KB page-aligned) plus the
tablet-compatibility milestones (system clipboard, open-with/share intents, flush on
suspend, back-gesture closes dialog, trackpad wheel + pen hover, stylus pressure for
PhotoCraft, PhotoCraft crash-recovery autosave).

A signed APK for the current state is attached to the
[v2026.10.30 release](https://github.com/physicistlk/photocraft-android/releases/tag/v2026.10.30).

See the upstream `kanaaka` repository for the full port notes and verification details.
