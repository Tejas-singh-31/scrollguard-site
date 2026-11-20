# ScrollGuard (Native Android App)

> **Official Landing Page & Waitlist:** [https://tejas-singh-31.github.io/scrollguard-site/](https://tejas-singh-31.github.io/scrollguard-site/)

ScrollGuard is an independent, distraction-free native Android app (Kotlin, Gradle, minSdk 26, targetSdk 34) that wraps the Instagram web client in a secure, full-featured WebView and removes Reels, algorithmic short-form video autoplay, and endless scrolling traps.

It contains **NO Meta code**, **NO Instagram proprietary assets or logos**, and is built from the ground up as a clean-room client.

---

## Architecture & Reels Neutralization

ScrollGuard uses a 3-layer defensive filter to neutralize short-form video doomscrolling:

1. **Navigation Chokepoint (Layer 1)**:
   - `WebViewClient.shouldOverrideUrlLoading` and `shouldInterceptRequest` intercept any URL whose path starts with `/reels`, `/reel/`, or `/explore/reels`, cancelling the request and redirecting to the Home feed (`https://www.instagram.com/`).
   - SPA navigation hook: Injected script hooks `history.pushState`, `history.replaceState`, and `popstate` to block client-side route transitions into Reels.
2. **Declarative CSS Injection (Layer 2)**:
   - Injected on `onPageFinished` and route mutations to permanently hide the bottom navigation Reels item (`a[href^='/reels']`), profile Reels tab, and search shelves.
3. **DOM MutationObserver (Layer 3)**:
   - `assets/hide_reels.js` continuously monitors DOM mutations to hide feed posts that are video reels, "Suggested Reels" trays, and inline carousels.
   - **Direct Messages (DMs)**: If a shared reel link/card is received in a chat, ScrollGuard replaces the video player card with a static placeholder (`🛡️ Reel hidden by ScrollGuard`) and stops autoplay.
4. **Configurable Selectors**:
   - All DOM selectors are centralized in `assets/selectors.json` so they can be updated quickly without modifying Kotlin code.

---

## Preserved Features

The app preserves all core social features:
- **Authentication**: Secure login via web interface; cookies persist across app restarts using `CookieManager`.
- **Feed**: Full photo posts, multi-photo carousels, and text captions.
- **Direct Messages**: Read, reply, send messages, and share photos.
- **Stories**: View stories from friends and accounts you follow.
- **Search & Explore**: Search for accounts, tags, and explore static posts (without Reels).
- **Profile & Notifications**: Full profile viewing and activity notifications.
- **Native Ergonomics**:
  - Pull-to-refresh (`SwipeRefreshLayout`).
  - Edge-to-edge system dark/light theme integration.
  - Hardware back button history navigation.
  - Camera and photo upload support (`WebChromeClient.onShowFileChooser`).
  - Strict HTTPS enforcement (`network_security_config.xml`) and safe SSL error handling (`handler.cancel()`).

---

## Build & Installation Guide

### Prerequisites
- JDK 17 or higher
- Android SDK with platform `android-34` and build-tools `34.0.0`

### 1. Build via Gradle
```bash
# Run unit tests
./gradlew test

# Build debug APK
./gradlew assembleDebug

# Build signed release APK
./gradlew assembleRelease
```

### 2. Release Signing Configuration
Copy `keystore.properties.example` to `keystore.properties` (gitignored):
```properties
storeFile=release.jks
storePassword=your_store_password
keyAlias=your_key_alias
keyPassword=your_key_password
```
If `keystore.properties` is absent, the build automatically falls back to debug signing to ensure builds remain reproducible.

### 3. Installation
#### A. Via ADB
Connect your phone with USB debugging enabled:
```bash
# Check device connection
adb devices

# Install release APK
adb install -r app/build/outputs/apk/release/app-release.apk
```

#### B. Via Sideloading
1. Transfer `app-release.apk` to your phone via USB cable, Google Drive, or local file transfer.
2. In your Android file manager, tap the APK and choose **Install** (enable "Install unknown apps" permission for your file manager if prompted).

---

## Known Limitations
1. **Web Interface Differences**: Instagram's mobile web client has slight differences compared to the official native app (e.g. advanced camera filters, live broadcasting, and draft reels editing are not supported on web).
2. **DOM Selectors Maintenance**: When Instagram updates its frontend class names or markup, selectors in `app/src/main/assets/selectors.json` may need minor updates.

---

## Disclaimer
ScrollGuard is an independent open-source project. Not affiliated with, authorized, maintained, sponsored, or endorsed by Instagram, Meta Platforms, Inc., or any of its affiliates.
