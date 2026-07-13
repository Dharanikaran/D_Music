# Plug — Android Project

This folder is a complete, ready-to-build Android project for the **Plug** music app.
It wraps the app (`www/index.html` — the same file from your chat) in a native Android
shell using [Capacitor](https://capacitorjs.com), so it can be compiled into a real
`.apk` you can install on a phone.

I can't produce a compiled `.apk` binary directly in this environment — building one
requires the Android SDK and Gradle, which need network access to Google's and
Gradle's servers that this sandbox doesn't have. Below are a few ways to actually
get the APK — **Option A needs no software installed on your computer at all** (it
builds in the cloud via GitHub); Options B and C use Android Studio or the command
line locally.

---

## Option A — Build in the cloud with GitHub Actions (no install needed)

This repo already includes a GitHub Actions workflow (`.github/workflows/build-apk.yml`)
that builds the APK automatically using a cloud machine with the Android SDK
pre-installed. You just need a free GitHub account.

**Important — how to upload this folder correctly:**
- Do **not** drag the `.zip` file itself into GitHub — GitHub stores zip files as-is,
  it does not extract them, so the workflow won't find your project inside it.
- First, **unzip** `PlugApp-Android-Project.zip` on your computer.
- This project intentionally does **not** include a `node_modules` folder (the
  workflow installs it automatically via `npm install`), which keeps it to about
  120 files — small enough for GitHub's drag-and-drop uploader to handle in one go.
  If you're using the folder from an earlier version of this project that *does*
  have `node_modules` in it, delete that folder first — it's the main reason
  uploads fail or only partially go through.

Steps:

1. Go to [github.com/new](https://github.com/new) and create a new repository
   (public or private, doesn't matter). **Don't** check "Add a README file" —
   leave it empty.
2. On the new repo's page, click **uploading an existing file**.
3. Open the unzipped `PlugApp` folder on your computer, select **everything inside
   it** (all files and subfolders — `www`, `android`, `.github`, `package.json`,
   etc.), and drag that selection onto the GitHub upload page. Drag the *contents*
   of the folder, not the `PlugApp` folder itself, so the files land at the root
   of the repo (GitHub should show `www/`, `android/`, `.github/` etc. directly,
   not nested inside another `PlugApp/` folder).
4. Scroll down and click **Commit changes**.
5. Click the **Actions** tab. A workflow called "Build Plug APK" will start
   automatically (takes a few minutes) — it runs `npm install` for you, so you
   don't need Node.js installed locally for this option.
6. When it finishes (green checkmark), click into that workflow run, scroll to
   **Artifacts**, and download **plug-debug-apk** — that's a zip containing your
   `app-debug.apk`.
7. Transfer the APK to your phone (email it to yourself, Google Drive, USB, etc.)
   and tap it to install — you'll need to allow "install from unknown sources" the
   first time.

If the web uploader still struggles (some browsers cap how many files can be
dragged at once), the most reliable option is **GitHub Desktop**
(desktop.github.com) — a free point-and-click app. Install it, choose
**Add Local Repository**, pick your unzipped `PlugApp` folder, then click
**Publish repository**. It handles folders of any size without the browser
upload limits.

This is the closest thing to "just give me the APK" without installing anything
locally — GitHub's servers do the actual compiling.

---

## Option B — Build with Android Studio (fully offline afterwards)

1. Install [Node.js](https://nodejs.org) (needed once, to install Capacitor's libraries)
   and [Android Studio](https://developer.android.com/studio) if you don't have them.
2. In a terminal, inside the unzipped `PlugApp` folder, run:
   ```bash
   npm install
   npx cap sync android
   ```
3. Open Android Studio → **Open** → select the `android/` folder inside this project
   (not the root `PlugApp` folder — the `android` subfolder specifically).
4. Let it sync (Android Studio will download any missing SDK components automatically).
5. Go to **Build → Build Bundle(s) / APK(s) → Build APK(s)**.
6. When it finishes, click **locate** in the notification, or find the file at:
   ```
   android/app/build/outputs/apk/debug/app-debug.apk
   ```
7. Copy that `.apk` to your phone and install it (you'll need to allow "install from
   unknown sources" the first time).

This produces a **debug APK** — perfect for installing on your own device to test.

---

## Option C — Build from the command line

Requires [Node.js](https://nodejs.org), a JDK (17 or 21), and the Android SDK
command-line tools installed, with `ANDROID_HOME`/`ANDROID_SDK_ROOT` set.

```bash
npm install
npx cap sync android
cd android
./gradlew assembleDebug          # macOS/Linux
gradlew.bat assembleDebug        # Windows
```

The APK will be at:
```
android/app/build/outputs/apk/debug/app-debug.apk
```

Install it on a connected device/emulator with:
```bash
adb install app/build/outputs/apk/debug/app-debug.apk
```

---

## What's in this folder

```
PlugApp/
├── .github/workflows/build-apk.yml  ← the GitHub Actions build (Option A)
├── www/
│   └── index.html          ← the Plug app itself (your HTML/CSS/JS file)
├── android/                ← full native Android Studio / Gradle project
│   ├── app/
│   │   ├── src/main/AndroidManifest.xml   ← app permissions & config
│   │   └── src/main/res/values/strings.xml ← app name ("Plug")
│   ├── build.gradle
│   └── gradlew / gradlew.bat              ← command-line build scripts
├── capacitor.config.ts     ← app ID (com.plug.musicapp), name, web folder
└── package.json            ← run `npm install` to generate node_modules locally
```

The app ID is `com.plug.musicapp` and the app name is `Plug`. Both can be changed —
see "Renaming the app" below.

> **Note:** `node_modules/` isn't included in this zip — it's regenerated by running
> `npm install` (Option A's GitHub Actions workflow does this automatically; for
> Options B/C you run it yourself once). This keeps the project small enough to
> upload easily.

---

## Making a release APK (for publishing / sharing widely)

Debug APKs are signed with a temporary debug key and are fine for testing, but
Google Play and most people installing the app expect a signed **release** build:

```bash
cd android
./gradlew assembleRelease
```

This needs a signing key. If you don't have one yet:
```bash
keytool -genkey -v -keystore plug-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias plug
```
Then configure signing in `android/app/build.gradle` (Android Studio's
**Build → Generate Signed Bundle / APK** wizard will do this for you interactively —
that's the easiest path).

---

## Editing the app

All the app's actual code lives in **`www/index.html`** — it's the same single-file
HTML/CSS/JS app from our conversation. To make changes:

1. Edit `www/index.html`.
2. Re-copy the web assets into the Android project:
   ```bash
   npx cap sync android
   ```
3. Rebuild the APK using any of the options above.

---

## Renaming the app or app ID

- **App name** shown under the icon: edit `android/app/src/main/res/values/strings.xml`
  (`app_name` and `title_activity_main`).
- **App ID** (package name, e.g. for the Play Store): edit `appId` in
  `capacitor.config.ts`, then update `applicationId` and `namespace` in
  `android/app/build.gradle` to match, and rename the Java package folder under
  `android/app/src/main/java/`.

---

## Permissions already configured

`AndroidManifest.xml` includes:
- `READ_MEDIA_AUDIO` (Android 13+) and `READ_EXTERNAL_STORAGE` (older Android) —
  so the in-app "Allow Access" flow can read music files from the device.
- `INTERNET` — added by Capacitor by default (used for the Google Fonts used in the
  UI; the app works fine offline aside from font loading falling back to system fonts).

The in-app folder picker (the "Allow Access" button) uses the device's native file/folder
picker, which is how Android grants storage access to apps sandboxed this way — you'll
see the real Android picker UI when you tap it.

---

## App icon / splash screen

This project currently uses Capacitor's default icon and splash screen. To customize:
```bash
npm install -D @capacitor/assets
npx capacitor-assets generate
```
(Provide your own `icon.png` / `splash.png` in an `assets/` folder first — see
[Capacitor's asset docs](https://capacitorjs.com/docs/guides/splash-screens-and-icons).)
