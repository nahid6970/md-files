# Install Rust with the GNU toolchain on Windows

This setup uses Rust's GNU toolchain instead of MSVC. Choose it if you do not need to link Rust programs against libraries built with MSVC.

## What this setup installs

- **rustup** — installs and manages Rust toolchains.
- **MinGW-w64** (`mingw-msvcrt`) — provides the GNU linker and related tools needed by the Windows GNU toolchain.
- **Stable GNU Rust toolchain** — the stable Rust compiler targeting `x86_64-pc-windows-gnu`.

## Prerequisite

Install [Scoop](https://scoop.sh/) if it is not already available in PowerShell.

## Install

Run these commands in PowerShell, in order:

```powershell
scoop install rustup
scoop install mingw-msvcrt
rustup toolchain install stable-x86_64-pc-windows-gnu
rustup default stable-x86_64-pc-windows-gnu
```

The first two commands install the Rust manager and GNU linker tools. The last two install and select the stable GNU Rust toolchain as the default.

## Verify

```powershell
rustup show active-toolchain
rustc --version
cargo --version
```

The active toolchain should be `stable-x86_64-pc-windows-gnu`.

## Trade-off

This configuration avoids the Microsoft C++ Build Tools. Rust projects that need to link against libraries built with MSVC may not work with this toolchain. The GNU toolchain includes a GCC-based linker and is described as being under 400 MB; actual download size can vary.

## Building the Tempo GPUI app on Windows

The Tempo project at `D:\speed` uses GPUI for its native Rust interface. The repository pins `nightly-x86_64-pc-windows-gnu` in `rust-toolchain.toml` because its `dll-syringe` dependency needs nightly Rust. In the project directory, `rustup show active-toolchain` should report that pinned nightly; the stable setup above remains useful for other Rust projects.

Install the x64 and i686 MinGW compilers. The app and x64 hook use x64 MinGW; the injected 32-bit game hook needs i686 MinGW. For the 32-bit compiler, install MSYS2 and use its MINGW32 shell:

```powershell
winget install --id MSYS2.MSYS2 -e
```

```bash
pacman -Syu
pacman -S --needed mingw-w64-i686-gcc
```

For release builds, GPUI compiles Windows shaders with `fxc.exe`. Install a Windows SDK that includes it, then set `GPUI_FXC_PATH`. The verified path on this machine is:

```powershell
$env:GPUI_FXC_PATH = "C:\Program Files (x86)\Windows Kits\10\bin\10.0.18362.0\x64\fxc.exe"
```

This setting lasts for the current PowerShell session. If the SDK is installed elsewhere, find the x64 compiler with `Get-ChildItem 'C:\Program Files (x86)\Windows Kits\10\bin' -Filter fxc.exe -Recurse`. See the [Windows SDK downloads](https://learn.microsoft.com/en-us/windows/apps/windows-sdk/downloads) and [GPUI Windows shader build script](https://github.com/zed-industries/zed/blob/main/crates/gpui_windows/build.rs).

From `D:\speed`, put x64 MinGW before MSYS2 MINGW32 on `PATH`, then build the hook DLLs and run the app:

```powershell
$env:Path = "C:\Users\nahid\scoop\apps\mingw-msvcrt\current\bin;C:\msys64\mingw32\bin;$env:Path"
npm run build:hook
npm run dev
```

To build a standalone release app, set `GPUI_FXC_PATH` in the same PowerShell session and run:

```powershell
npm run build:windows
```

The executable is `src-tauri\target\release\tempo-speed-control.exe`. Keep it beside the x86/x64 hook DLLs and `.current` pointer files. GPUI uses native Win32 windows and DirectWrite on Windows, so Tempo does not need Tauri, WebView2, Vite, or an HTML frontend. Cargo downloads the GPUI crates on the first build.
