# Insta Access

A lightweight Android WebView wrapper around the **official Instagram website**
(`https://www.instagram.com/`). Nothing is faked, mirrored or proxied — the app
simply hosts the real Instagram site in a well-configured, full-screen WebView.

## What it does

| Requirement | How it's met |
|---|---|
| Open the official Instagram website | `MainActivity` loads `https://www.instagram.com/` only |
| Real website in a WebView | Standard `android.webkit.WebView`, no custom UI around it |
| JavaScript + DOM storage | `javaScriptEnabled = true`, `domStorageEnabled = true`, `databaseEnabled = true` |
| Cookies & sessions | `CookieManager` accepts first- and third-party cookies and flushes them on pause/destroy so logins persist |
| In-app navigation | `WebViewClient` keeps Instagram (and its auth hosts) inside the WebView; other links open in the browser |
| File uploads | `WebChromeClient.onShowFileChooser` + `ActivityResultContracts.StartActivityForResult` (images, video, audio, PDF, multiple selection) |
| Media playback | `mediaPlaybackRequiresUserGesture = false`, hardware acceleration on |
| Android back button | `OnBackPressedCallback` walks WebView history before leaving the app |
| Splash screen | `androidx.core:core-splashscreen` (Android 6.0–11 compat + native Android 12+ splash), stays visible until the first page renders |
| Icon & name | Instagram-style gradient adaptive icon, app name **Insta Access** |
| Full-screen & responsive | Edge-to-edge dark theme, `adjustResize`, no chrome around the page |
| No fake login | The login page users see is Instagram's own page |
| No credential collection | No account, no server, no network code of its own — only the WebView talks to Instagram |
| No security bypass | Instagram's site, scripts and checks run untouched |

## Requirements

- Android Studio Koala (or newer) — bundles JDK 17 and the Android SDK 35
- Android 6.0+ device or emulator (`minSdk 24`, `targetSdk 35`)

## Build an APK

### Option A — Android Studio (easiest)

1. Unzip this project.
2. In Android Studio choose **File → Open** and select the `insta-access`
   folder (the one containing `settings.gradle.kts`).
3. Let Gradle sync finish (first sync downloads dependencies).
4. **Build → Build Bundle(s) / APK(s) → Build APK(s)**.
5. The debug APK appears in `app/build/outputs/apk/debug/app-debug.apk`.
   Click **Locate** in the notification to find it.

### Option B — Command line

```bash
cd insta-access

# Debug APK
./gradlew assembleDebug
# -> app/build/outputs/apk/debug/app-debug.apk

# Release APK (unsigned / signed per your keystore config)
./gradlew assembleRelease
# -> app/build/outputs/apk/release/app-release.apk
```

On Windows use `gradlew.bat` instead of `./gradlew`.

### Install on a device

```bash
adb install app/build/outputs/apk/debug/app-debug.apk
```

or enable USB debugging and press **Run ▶** in Android Studio.

### Signing a release build (Play Store / distribution)

Create a keystore once:

```bash
keytool -genkeypair -v -keystore insta-access.jks \
  -keyalg RSA -keysize 2048 -validity 10000 -alias instaaccess
```

Then in Android Studio: **Build → Generate Signed Bundle / APK → APK** and
select the keystore and passwords. Or add the keystore config to
`app/build.gradle.kts` under `buildTypes.release` (`signingConfig`).

## Project layout

```
insta-access/
├── settings.gradle.kts
├── build.gradle.kts
├── gradle.properties
├── gradle/
│   ├── libs.versions.toml          # dependency versions
│   └── wrapper/                    # Gradle wrapper (pin to Gradle 8.9)
├── gradlew / gradlew.bat
└── app/
    ├── build.gradle.kts            # app module config (minSdk 24, targetSdk 35)
    ├── proguard-rules.pro
    └── src/main/
        ├── AndroidManifest.xml
        ├── java/com/instaaccess/app/MainActivity.kt   # all WebView logic
        └── res/
            ├── layout/activity_main.xml              # full-screen WebView + pull-to-refresh
            ├── drawable/ic_launcher_background.xml   # gradient icon background
            ├── drawable/ic_launcher_foreground.xml   # camera glyph
            ├── mipmap-*/                             # launcher icons (all densities)
            └── values/                               # strings, colors, themes (splash)
```

## Behavior notes

- **Navigation whitelist** — top-level navigations are allowed only to
  `instagram.com`, its API/help hosts, and the `facebook.com` hosts Instagram
  uses for sign-in. Any other web link opens in the user's browser instead.
- **Camera / microphone** — when Instagram itself asks (e.g. recording), the
  app requests the matching Android permission and only then grants it to the
  page. Location is intentionally never granted.
- **Downloads** — links that trigger downloads use the system
  `DownloadManager`.
- **Pull to refresh** — swipe down on the feed reloads the page.
- **Session persistence** — cookies/localStorage flush on every pause and are
  restored with `webView.saveState/restoreState` across process restarts.

## Privacy & policy statement

Insta Access performs no analytics, telemetry, tracking or account creation.
It does not access, copy or transmit Instagram credentials, cookies or session
tokens anywhere beyond the WebView's own on-device storage. If you publish it,
use this wording in your store listing:

> Insta Access is a browser wrapper for the official Instagram website. It
> does not collect, store or transmit any personal data on its own.
