# Android Launcher Icon Setup

This note documents the launcher icon setup that worked for the Tab Saver Android app. It uses PNG launcher assets in density-specific `mipmap` folders and points the app manifest directly at them.

## Resource layout

Android selects the closest density asset for the device.

| Resource folder | PNG size |
|---|---:|
| `mipmap-mdpi` | 48 × 48 px |
| `mipmap-hdpi` | 72 × 72 px |
| `mipmap-xhdpi` | 96 × 96 px |
| `mipmap-xxhdpi` | 144 × 144 px |
| `mipmap-xxxhdpi` | 192 × 192 px |

Each folder contains `ic_tab_saver_flat.png`. Keep the image square and center the mark with enough padding for launcher masks. This project uses the Chrome extension mark centered on a dark background at about 62% of the icon canvas.

## Manifest

Set both icon attributes on the `<application>` element:

```xml
<application
    android:icon="@mipmap/ic_tab_saver_flat"
    android:roundIcon="@mipmap/ic_tab_saver_flat"
    ...>
```

The resource name must match the PNG basename without `.png`. Android documents `android:icon` as the app's icon and recommends storing launcher assets in the density-specific `mipmap` directories. [App icon guide](https://developer.android.com/studio/write/create-app-icons?hl=en) · [Application manifest element](https://developer.android.com/guide/topics/manifest/application-element.html)

## Generate density assets

Use Python with Pillow to resize the source mark into each density bucket. For this project, the source is `C:\Users\nahid\delta\extension\tab_saver\icons\icon128.png`. Scale the mark to around 62% of each square canvas, center it, and composite it onto a dark background. Save the results as `ic_tab_saver_flat.png` in the five `mipmap-*` folders above.

## Build and verify

From PowerShell at the project root:

```powershell
.\gradlew.bat :app:assembleDebug
```

Inspect the icon resource packaged in the APK:

```powershell
$aapt = Join-Path $env:ANDROID_HOME 'build-tools\35.0.0\aapt.exe'
& $aapt dump badging app\build\outputs\apk\debug\app-debug.apk |
    Select-String 'package:|application: label|application-icon-'
```

For this raster setup, the reported `application-icon` path should end in `ic_tab_saver_flat.png`. If it reports an adaptive XML icon, check the manifest resource name and any `mipmap-anydpi-v26` resources using that same name.

The debug APK is written to `app\build\outputs\apk\debug\app-debug.apk`.

## Updating the installed app

Keep the same `applicationId` and signing key to install an update without clearing local data. Increment `versionCode` for each release. If only a home-screen shortcut shows the old icon, remove that shortcut and add the app again from the app drawer. Avoid uninstalling first because uninstalling clears the app's local data.

## Adaptive icon alternative

Adaptive icons use separate foreground and background layers in `mipmap-anydpi-v26`; density-specific mipmaps are still required for older Android versions. If using adaptive icons, verify both layers and the manifest reference. This app uses direct PNG mipmaps because that setup displayed the desired icon consistently on the target device.
