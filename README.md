# Daily Image Rotator — Android App

A clean Material Design Android app that automatically rotates through your uploaded images once every 24 hours using Android WorkManager.

---

## Features

| Feature | Details |
|---------|---------|
| 📸 Image Upload | Pick multiple photos from the gallery at once |
| 🔄 Daily Rotation | Automatically advances to the next image every 24 h via WorkManager |
| ♾️ Continuous Cycle | Wraps back to the first image after the last |
| 🖼️ Full-Screen Viewer | Tap the main image to open a pinch-to-zoom full-screen view |
| 🗂️ Thumbnail Strip | Horizontal gallery of all uploaded images; tap any to jump to it |
| 🗑️ Delete Images | Long-tap the × on any thumbnail to remove an image |
| 🔕 Offline | 100% offline; no internet permission required |
| 📦 Image Compression | Images are automatically resized (max 1920 px) and JPEG-compressed at 80% quality |
| 🔋 Battery Friendly | WorkManager respects Doze mode and device constraints |

---

## Project Structure

```
DailyImageRotator/
├── app/
│   ├── src/main/
│   │   ├── AndroidManifest.xml
│   │   ├── java/com/dailyimagerotator/
│   │   │   ├── data/
│   │   │   │   └── ImagePreferences.kt       # SharedPreferences wrapper
│   │   │   ├── ui/
│   │   │   │   ├── MainActivity.kt           # Main screen
│   │   │   │   ├── MainViewModel.kt          # ViewModel & business logic
│   │   │   │   ├── FullScreenActivity.kt     # Full-screen pinch-zoom viewer
│   │   │   │   └── ThumbnailAdapter.kt       # RecyclerView adapter
│   │   │   ├── utils/
│   │   │   │   ├── ImageUtils.kt             # Compress & copy images
│   │   │   │   └── PermissionUtils.kt        # Runtime permission helpers
│   │   │   └── worker/
│   │   │       ├── ImageRotationWorker.kt    # WorkManager worker
│   │   │       ├── WorkManagerScheduler.kt   # Schedules the periodic work
│   │   │       └── BootReceiver.kt           # Re-schedules after reboot
│   │   └── res/
│   │       ├── layout/
│   │       │   ├── activity_main.xml
│   │       │   ├── activity_fullscreen.xml
│   │       │   └── item_thumbnail.xml
│   │       ├── drawable/  (vector icons + ripple)
│   │       ├── values/    (colors, strings, themes)
│   │       └── xml/       (backup rules, file paths)
│   ├── build.gradle.kts
│   └── proguard-rules.pro
├── build.gradle.kts
├── settings.gradle.kts
└── gradle.properties
```

---

## Requirements

| Requirement | Version |
|-------------|---------|
| Android Studio | Hedgehog 2023.1.1 or newer |
| JDK | 17 (bundled with Android Studio) |
| Gradle | 8.2 (downloaded automatically) |
| Android SDK | API 34 (compileSdk) |
| Min Android | API 24 (Android 7.0 Nougat) |

---

## Build & Run Instructions

### Option A — Android Studio (Recommended)

1. **Open the project**
   - Launch Android Studio → **File → Open** → select the `DailyImageRotator` folder.

2. **Sync Gradle**
   - Android Studio will prompt "Gradle files have changed". Click **Sync Now**.
   - This downloads all dependencies (~100 MB on first run).

3. **Connect a device or start an emulator**
   - Use a physical device with USB Debugging enabled, **or**
   - Launch an AVD via **Device Manager** → **Create Device** (API 24+).

4. **Run the app**
   - Click the green ▶ **Run** button or press `Shift+F10`.

---

### Option B — Build a Debug APK from the command line

```bash
# macOS / Linux
cd DailyImageRotator
chmod +x gradlew
./gradlew assembleDebug

# Windows
cd DailyImageRotator
gradlew.bat assembleDebug
```

The APK is generated at:
```
app/build/outputs/apk/debug/app-debug.apk
```

Install on a connected device:
```bash
adb install app/build/outputs/apk/debug/app-debug.apk
```

---

### Option C — Build a Release APK

1. Generate a keystore (one-time):
   ```bash
   keytool -genkey -v -keystore my-release-key.jks \
     -keyalg RSA -keysize 2048 -validity 10000 \
     -alias daily-image-rotator
   ```

2. Add signing config to `app/build.gradle.kts`:
   ```kotlin
   signingConfigs {
       create("release") {
           storeFile = file("../my-release-key.jks")
           storePassword = "YOUR_STORE_PASSWORD"
           keyAlias = "daily-image-rotator"
           keyPassword = "YOUR_KEY_PASSWORD"
       }
   }
   buildTypes {
       release {
           signingConfig = signingConfigs.getByName("release")
           // existing minify settings ...
       }
   }
   ```

3. Build:
   ```bash
   ./gradlew assembleRelease
   ```
   Output: `app/build/outputs/apk/release/app-release.apk`

---

## How the Daily Rotation Works

1. **On first upload** — `ImagePreferences.initRotationTime()` records the current timestamp.
2. **Every time the app opens** — `MainViewModel.loadImages()` calls `ImagePreferences.shouldRotate()`. If ≥ 24 h have passed, it calls `rotateToNext()`, which increments the index and saves the new timestamp.
3. **In the background** — `WorkManagerScheduler` enqueues a `PeriodicWorkRequest` with a 24-hour interval. `ImageRotationWorker.doWork()` runs the same rotate logic and fires a local notification.
4. **After reboot** — `BootReceiver` re-enqueues the periodic work so it survives device restarts.

---

## Permissions

| Permission | Purpose |
|------------|---------|
| `READ_MEDIA_IMAGES` (API 33+) | Pick images from the gallery |
| `READ_EXTERNAL_STORAGE` (API ≤ 32) | Pick images on older Android |
| `RECEIVE_BOOT_COMPLETED` | Re-schedule work after reboot |
| `POST_NOTIFICATIONS` (API 33+) | Show daily rotation notification |
| `FOREGROUND_SERVICE` | Required by WorkManager on some OEMs |

All permissions are requested at runtime; the app degrades gracefully if denied.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Gradle sync fails | Check Android SDK is installed: **SDK Manager → SDK Platforms → API 34** |
| PhotoView not found | Ensure JitPack is in `settings.gradle.kts` repositories |
| Images not rotating in background | Disable battery optimisation for the app: **Settings → Battery → App launch** |
| Build error "duplicate class" | Run `./gradlew clean` then rebuild |
