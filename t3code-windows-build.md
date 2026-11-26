# Build T3 Code for Windows (x64)

This is the end-to-end record of setting up and building the T3 Code source checkout at `C:\Users\nahid\t3code` on this PC.

## PC and result

- CPU: AMD Ryzen 5 3400G, 4 cores / 8 threads
- RAM: about 14 GB
- Free disk space when checked: about 278 GB
- Installer produced: `C:\Users\nahid\t3code\release\T3-Code-0.1.13-x64.exe` (about 203 MB)
- The installer is an unsigned Windows `.exe` (NSIS), not an MSI.

## What was needed

- Git, to clone the repository.
- Vite+ (`vp`), which provides the project's pinned Node.js and package manager. The repo pins Node `26.13.1` and pnpm `12.14.0`.
- Rust (`rustup`, `rustc`, `cargo`) and the MSVC Rust target `x86_64-pc-windows-msvc`.
- Python 3 (Python 3.14.5 was present when checked).
- Visual Studio Build Tools 2022, with the x64/x86 MSVC compiler, Windows SDK, and x64 Spectre libraries. The full Visual Studio IDE and the broad C++ workload were not selected.

MSYS2/MinGW `g++` was present, but it does not replace MSVC for this project: the Windows build uses Rust's MSVC target and checks for MSVC, the Windows SDK, and Spectre libraries.

## Setup commands

### 1. Clone the repository

The checkout already existed at `C:\Users\nahid\t3code`. For a fresh checkout, the repo's Windows guidance recommends enabling Git long paths:

```powershell
git clone -c core.longpaths=true https://github.com/pingdotgg/t3code.git C:\Users\nahid\t3code
cd C:\Users\nahid\t3code
```

### 2. Install Vite+

```powershell
irm https://vite.plus/ps1 | iex
```

If `vp` is not recognized, reopen PowerShell. In this setup, `vp.exe` was found at `%LOCALAPPDATA%\vite-plus\bin\vp.exe` and Node was `v26.13.1`.

### 3. Install the minimal Visual Studio Build Tools components

This is the command used. It installs only the selected compiler, SDK, and Spectre component plus their installer-required dependencies; it does not install the Visual Studio IDE:

```powershell
winget install --exact --id Microsoft.VisualStudio.2022.BuildTools `
  --accept-package-agreements --accept-source-agreements `
  --override "--wait --quiet --norestart --add Microsoft.VisualStudio.Component.VC.Tools.x86.x64 --add Microsoft.VisualStudio.Component.Windows11SDK.26100 --add Microsoft.VisualStudio.Component.VC.Runtimes.x86.x64.Spectre"
```

The install was verified: `cl.exe`, the Windows SDK, and the x64 Spectre library directory were present.

### 4. Add the Rust MSVC target and install repo dependencies

The PC initially had Rust and Cargo, but only the GNU Rust target. Add the MSVC target required by this build, then install the JavaScript workspace dependencies:

```powershell
cd C:\Users\nahid\t3code
rustup target add x86_64-pc-windows-msvc
vp i
```

## Build command

The ordinary x64 Windows installer command is:

```powershell
vp run dist:desktop:win:x64
```

The equivalent direct script command used during troubleshooting was:

```powershell
node scripts/build-desktop-artifact.ts --platform win --target nsis --arch x64
```

## Build issues and fixes

### `EBUSY` while processing `T3 Code (Beta).exe`

The first attempt failed late in packaging when `electron-fuses` tried to open the staged app executable and Windows reported `EBUSY` (file busy/locked). T3 Code Nightly was closed before retrying. The next attempt got further and reached payload validation, so the later 104-file error was a separate issue. The log did not prove what held the executable lock.

### Payload contained 104 files; limit was 80

The next attempt failed the repo's Windows payload size check. A diagnostic build with `--keep-stage` showed that the generated `resources\server-compile-cache` contained 38 separate files. The whole package then had 104 files, above the project's limit of 80.

For the successful local build, the project's `T3CODE_DESKTOP_UNTRUSTED_BUNDLE=true` mode was used to skip generating this optional cache. That reduced the payload to 66 files and allowed payload validation to pass. It also skips the packaged-bundle self-check, so use this mode only for a build from source you trust. Without the prepared server bytecode cache, first startup may be slower.

PowerShell command that succeeded:

```powershell
cd C:\Users\nahid\t3code
$env:T3CODE_DESKTOP_UNTRUSTED_BUNDLE = 'true'
node scripts/build-desktop-artifact.ts --platform win --target nsis --arch x64
```

The environment variable was set only for that PowerShell process; it was not saved permanently in Windows settings.

### Messages that were warnings, not the build failure

- “Platform-specific optional dependencies not bundled” listed packages for macOS, ARM, Linux, and other targets; those are not needed for this Windows x64 build.
- “No code signing certificate configured, signing is skipped” is expected for this local unsigned build. Windows may show a SmartScreen warning when running the installer.

## Output and installing

The successful build reported:

```text
C:\Users\nahid\t3code\release\T3-Code-0.1.13-x64.exe
C:\Users\nahid\t3code\release\T3-Code-0.1.13-x64.exe.blockmap
```

Run `T3-Code-0.1.13-x64.exe` to install the app. It is an installer, not a portable app to launch directly. The `.blockmap` file is an update-related sidecar, not the installer.

## Login, app data, and updates

- Sign into the coding agent/provider you want to use (such as Codex or Claude) through its normal setup. Those credentials are separate from building the app.
- T3 Connect sign-in is optional for local desktop use; it is used for remote access from a phone or browser.
- A local build uses the T3 app identity and may share app data with another installed T3 Code build. Do not assume it is isolated from T3 Code Nightly or another official build.
- The build script only adds a GitHub update feed when `T3CODE_DESKTOP_UPDATE_REPOSITORY` or `GITHUB_REPOSITORY` is set. A local build without one of those set likely will not auto-update; rebuild from newer source to update it.
