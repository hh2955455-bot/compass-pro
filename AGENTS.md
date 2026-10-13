# Task: package the web compass as an Android app

Turn the single-page compass in `www/index.html` into an installable Android app using Capacitor. Do not redesign or rewrite the UI. The goal is a debug APK that installs and runs without errors.

## What is already here

- `www/index.html`: the whole app (HTML, CSS and JS in one file). Arabic, right-to-left. Two modes:
  - Manual: drag the dial or move the slider.
  - Device sensor: the "حساس الجهاز" button listens to `deviceorientationabsolute` / `deviceorientation`. If no reading arrives within 3 seconds it falls back to manual mode and shows a message.
- `package.json`, `capacitor.config.json`: Capacitor 7 setup. App id `com.example.simplecompass`, name `بوصلة بسيطة`, web dir `www`.
- Capacitor serves the page from `https://localhost` inside the Android WebView, which is a secure context, so orientation events are allowed. No Android permission is needed for them.

## Steps

1. Check the toolchain: Node 20 or newer, JDK 21, Android SDK with platform tools (Android Studio installs these). Install what is missing. Ask the user only for things you cannot do yourself.
2. `npm install`
3. `npx cap add android`
4. `npx cap sync android`
5. Lock the activity to portrait in `android/app/src/main/AndroidManifest.xml` by adding `android:screenOrientation="portrait"` to the main `<activity>`. The heading math in `www/index.html` assumes portrait.
6. Build a debug APK: `cd android && ./gradlew assembleDebug` (`gradlew.bat` on Windows). Output: `android/app/build/outputs/apk/debug/app-debug.apk`.
7. Install and run on an emulator or a real device, then check the list below.

## Acceptance checks

- The build finishes with no errors.
- The app opens in portrait, in Arabic, and the degree readout is not hidden under the status bar or navigation bar. If it is, adjust the top and bottom padding in `www/index.html` (it already uses `env(safe-area-inset-*)`), or configure Capacitor's system bars.
- Manual mode: dragging the dial and moving the slider both change the degrees and the direction name.
- Sensor mode: tapping "حساس الجهاز" shows "الحساس يعمل..." and the dial follows the phone when it is rotated. On an emulator use Extended controls, Virtual sensors, Device pose.
- Tapping "يدوي" returns to manual mode and re-enables the slider.
- No errors in `adb logcat` or in the WebView console (`chrome://inspect`).

## If sensor events do not arrive in the WebView

Only then, add a small native Capacitor plugin that reads `Sensor.TYPE_ROTATION_VECTOR`, converts it with `SensorManager.getRotationMatrix` and `getOrientation` to a heading in degrees (0 to 360, clockwise from north), and sends it to the page. Feed it through the same code path that `onOrient` uses in `www/index.html` and keep the existing web path as the fallback.

## Rules

- Keep manual mode and the fallback messages. Do not remove error handling.
- Do not create a release keystore or signed release build without asking the user first. Never commit keystores.
- Optional, only if asked: a compass launcher icon and a release AAB.
