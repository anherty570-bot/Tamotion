# TA MOTION — Build notes

## What this is
TA MOTION is a mobile-first Android video editor rebranded from EffectCraft Pocket (duplexspy/effectscraft-unofficial-mobile-port), licensed MIT OR Apache-2.0 — modification and redistribution permitted.

## Architecture (preserved from EffectCraft Pocket)
- Pure Rust engine pre-compiled to WebAssembly (`app/src/main/assets/web/effectcraft_web_bg.wasm`, ~300 effects / 3009 params)
- Android wrapper: `MainActivity.java` + hardware-accelerated WebView serving bundled assets offline via `shouldInterceptRequest`
- Keyframes (Linear/Bezier/Hold), Graph Editor, layer/property/project model, effect control channel (JSON `engine.execute` / `ui.inspect`)
- No Rust toolchain needed to build the APK — the WASM is already compiled and bundled

## How to build the APK
Use GitHub Actions (free, no credit card): `.github/workflows/build.yml`
- Trigger: push to `main` or `workflow_dispatch`
- Ubuntu-latest (glibc) runner, JDK 17 (Temurin), Android SDK 35 / build-tools 35.0.0, Gradle 8.13
- Output: `app/build/outputs/apk/release/app-release.apk`, uploaded as artifact `tamotion-apk` (30-day retention)

## Why local build fails in this sandbox
This environment is Alpine Linux (musl libc). Android's AAPT2 is a native binary compiled for glibc and its daemon fails to start on musl (`AAPT2 daemon startup failed`). A glibc-based runner (GitHub Actions ubuntu-latest) is required. Do not attempt to fix by installing `libc6-compat` — AAPT2 needs the real glibc dynamic linker.

## Key files
- `app/src/main/java/ai/storyteller/tamotion/MainActivity.java` — WebView shell, asset serving, file import, crash recovery
- `app/src/main/assets/web/index.html` — WASM bootstrap, Vietnamese UI strings
- `app/src/main/assets/web/effectcraft_web_bg.wasm` — pre-built EffectCraft engine
- `app/src/main/res/values/strings.xml` — Vietnamese labels
- `build.gradle`, `settings.gradle`, `gradle.properties` — CI-safe (no hardcoded local paths)

## Signing
Release build uses `signingConfigs.debug` (debug keystore). For Play Store publishing, replace with a real release signing config and never commit signing passwords.
