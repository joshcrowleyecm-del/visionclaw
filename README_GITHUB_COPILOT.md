# VisionClaw - Android Studio Copilot Runbook

Goal: get the VisionClaw Android sample running on Windows, once and correctly, with minimal changes.

Scope: build and run only `samples/CameraAccessAndroid`.

Constraints:
- Windows machine
- Android Studio
- Authenticate to GitHub Packages to download Meta DAT Android SDK
- Do not mix non-Gradle Java modules with Android-Gradle modules
- Do not request tokens or keys in chat

## Phase 0 - Open the right project (avoid module mixups)
1. In Android Studio: `File -> Close Project`.
2. From Welcome screen, open folder:
   `visionclaw/samples/CameraAccessAndroid`
3. If prompted to link Gradle project:
   - Select `visionclaw/samples/CameraAccessAndroid/settings.gradle.kts`
   - Trust project and use Gradle wrapper.

Important: do not open repo root. Open `CameraAccessAndroid` directly.

## Phase 1 - Create `local.properties` (GitHub Packages auth for Meta DAT SDK)
1. Ensure file exists:
   `visionclaw/samples/CameraAccessAndroid/local.properties`
2. Ensure it contains exactly:

```properties
github_token=YOUR_GITHUB_PAT_WITH_read:packages
```

Rules:
- No quotes
- No spaces around `=`
- Do not commit this file

## Phase 2 - Create `Secrets.kt` (required)
1. Source template:
   `visionclaw/samples/CameraAccessAndroid/app/src/main/java/com/meta/wearable/dat/externalsampleapps/cameraaccess/Secrets.kt.example`
2. Create:
   `visionclaw/samples/CameraAccessAndroid/app/src/main/java/com/meta/wearable/dat/externalsampleapps/cameraaccess/Secrets.kt`
3. Set Gemini API key in `Secrets.kt`.
4. For Ground Zero baseline, keep OpenClaw and WebRTC disabled:

```kotlin
const val openClawHost = ""
const val openClawPort = 0
const val openClawHookToken = ""
const val openClawGatewayToken = ""
const val webrtcSignalingURL = ""
```

## Phase 3 - Fix invalid `org.gradle.java.home` (macOS path on Windows)
If Gradle error references a macOS JDK path like:
`/Applications/Android Studio.app/Contents/jbr/Contents/Home`

Remove `org.gradle.java.home` from:
1. `visionclaw/samples/CameraAccessAndroid/gradle.properties` (if present)
2. `C:\Users\<you>\.gradle\gradle.properties` (if present)

Then in Android Studio set:
`Settings -> Build, Execution, Deployment -> Build Tools -> Gradle -> Gradle JDK = Embedded JDK`

## Phase 4 - Remove non-Gradle module if present
If Android Studio reports compilation is not supported due to mixed non-Gradle and Android-Gradle modules (module `visionclaw`):

1. Open `File -> Project Structure -> Modules`
2. Select module `visionclaw`
3. Remove it with `-`
4. Keep only `CameraAccessAndroid`

This updates IDE module model only and should not delete source files.

## Phase 5 - Force Gradle resolve and sync
From terminal at `visionclaw/samples/CameraAccessAndroid` run:

```powershell
.\gradlew.bat --stop
.\gradlew.bat clean
.\gradlew.bat assembleDebug
```

Failure handling:
- `401 Unauthorized`: `github_token` missing/invalid or missing `read:packages`
- Cannot resolve `mwdat-core`: verify project was opened at `CameraAccessAndroid`, verify `local.properties` path/name/value

## Phase 6 - Run baseline (phone mode first)
1. Connect Android phone with USB debugging enabled.
2. Run app (`Shift+F10`).
3. In app: tap `Start on Phone`, then tap AI button to start Gemini Live.

Baseline success criteria: Gemini pipeline works without glasses.

## Phase 7 - Glasses mode (after phone mode works)
1. In Meta AI app:
   - `Settings -> App Info`
   - Tap app version 5 times
   - Enable Developer Mode
2. In VisionClaw app:
   - Tap `Start Streaming`
   - Tap AI button

## Phase 8 - Optional later: OpenClaw and WebRTC
Only enable after baseline is stable.

Notes:
- Configure OpenClaw host/port/token in `Secrets.kt` and/or in-app Settings.
- WebRTC requires signaling server.
- WebRTC and Gemini Live may conflict on audio device when used simultaneously.

## Error Output Contract
When any step errors, output only these 4 items:
1. Root cause
2. Exact file path to change
3. Exact lines to add/remove
4. Exact command to rerun
