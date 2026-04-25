---
name: wayland-app-icon-fix
description: "Diagnose and fix generic Wayland taskbar or dock icons caused by desktop-file and runtime app-ID mismatches on Linux, especially Electron or Chromium apps on KDE Plasma."
---

# Wayland App Icon Fix

## Overview

Use this skill when a Linux app launches and works, but the dock, taskbar, or app switcher shows a generic Wayland icon or fails to associate the running window with the correct launcher icon.

This is common with Electron and Chromium apps when the runtime desktop identity differs from the installed desktop file. Typical examples:

- desktop entry: `Foo.desktop`
- runtime identity: `Foo.bin`
- taskbar result: generic Wayland icon or duplicate launcher entries

Prefer local per-user overrides. Do not edit package-managed files in `/usr/share/applications`, `/opt`, or `/usr/bin` unless the user explicitly asks for a system-level fix.

## When To Use

- The app launches, but Plasma, GNOME, or another Wayland shell shows the wrong icon
- The launcher icon and the running-window icon do not group together
- The app is packaged as Electron or Chromium and installed from an RPM, DEB, AppImage, tarball, or custom wrapper

Do not use this skill first for:

- missing icon assets entirely
- broken `.desktop` syntax that prevents launcher discovery
- Flatpak or Snap packaging issues that are clearly sandbox-specific

## Core Principle

The main job is to identify the runtime desktop identity the compositor sees, compare it to the installed desktop file, and create the smallest local alias or wrapper needed so the desktop environment can match them.

## Workflow

### 1. Inspect the installed launcher

Read the installed desktop file if the path is known. Capture at least:

- `Name`
- `Exec`
- `Icon`
- `StartupWMClass`

If the path is not known, search common locations first:

- `~/.local/share/applications`
- `/usr/share/applications`

Validate the file with `desktop-file-validate` if available.

### 2. Identify the runtime desktop identity

On KDE Plasma Wayland, prefer a search-first flow that does not require the app to be focused.

Recommended order:

1. Search candidate windows with `kdotool`
2. Inspect each candidate with `kdotool getwindowclassname`, `kdotool getwindowname`, and `gdbus ... org.kde.KWin.getWindowInfo`
3. Only fall back to active-window inspection if search is ambiguous

Useful commands:

```bash
# Search candidates by likely runtime names
kdotool search Foo
kdotool search Foo.bin
kdotool search foo

# Inspect a candidate UUID returned by kdotool
kdotool getwindowclassname "{uuid}"
kdotool getwindowname "{uuid}"
gdbus call --session \
  --dest org.kde.KWin \
  --object-path /KWin \
  --method org.kde.KWin.getWindowInfo "{uuid}"

# Fallback: only if needed, inspect the active window
gdbus call --session \
  --dest org.kde.KWin \
  --object-path /KWin \
  --method org.kde.KWin.queryWindowInfo
```

Important:

- Title search can be noisy because terminal titles and chat titles may mention the app name
- Prefer runtime class matches like `Foo.bin` over title matches like `Foo`
- On Plasma, the most useful fields are usually `desktopFile`, `resourceClass`, and `resourceName`

### 3. Compare launcher vs runtime identity

Look for mismatches like:

- launcher file `Foo.desktop`, runtime `desktopFile='Foo.bin'`
- launcher `StartupWMClass=Foo`, runtime `resourceClass='foo'`
- wrapper script launches a binary whose basename differs from the desktop file name

If the launcher already matches the runtime identity, the issue may be a stale pinned entry or icon asset problem instead of an app-ID mismatch.

### 4. Apply the smallest local fix

Default to a per-user override. Prefer these locations:

- wrapper: `~/.local/bin/<app>-desktop`
- desktop override: `~/.local/share/applications/<Launcher>.desktop`
- desktop alias: `~/.local/share/applications/<RuntimeId>.desktop`

Use one or both of these techniques as needed:

#### Technique A: wrapper that sets `CHROME_DESKTOP`

Use this for Electron or Chromium apps launched by a shell script or desktop entry.

Example:

```bash
#!/bin/bash
export CHROME_DESKTOP="Foo.bin.desktop"
exec "/opt/Foo/Foo" "$@"
```

#### Technique B: hidden desktop-file alias matching the runtime ID

Create a desktop file whose basename matches the runtime identity exactly.

Example:

```ini
[Desktop Entry]
Name=Foo
Exec=/home/user/.local/bin/foo-desktop %U
Terminal=false
Type=Application
Icon=Foo
StartupWMClass=Foo.bin
Comment=Foo desktop app
Categories=Development;
NoDisplay=true
```

Keep the visible launcher file too, usually with the same `Exec`, `Icon`, and `StartupWMClass`.

### 5. Refresh the desktop environment

After writing local overrides:

- mark wrappers executable
- validate desktop files
- rebuild desktop caches when available

Examples:

```bash
chmod +x ~/.local/bin/foo-desktop
desktop-file-validate ~/.local/share/applications/Foo.desktop
desktop-file-validate ~/.local/share/applications/Foo.bin.desktop
kbuildsycoca6
```

### 6. Verify the result

Ask the user to:

1. fully quit the app
2. relaunch it from the app launcher, not an old pinned entry
3. re-pin the running app if the shell was holding onto the old association

If the icon is still wrong, re-check the actual runtime identity. Do not assume the first guess was correct.

## Safety Rules

- Prefer local fixes in `~/.local/share/applications` and `~/.local/bin`
- Do not patch `/usr/share/applications` or `/opt/...` for package-managed apps unless the user explicitly wants a system-wide change
- Do not delete or replace the vendor desktop file
- Do not change unrelated fields in the desktop entry if a smaller fix works
- If the file is managed by a dotfile system like chezmoi, edit the source rather than the live file when appropriate

## Heuristics

- Electron apps often expose a runtime ID based on the binary name, such as `Foo.bin`
- If the installed launcher runs a shell script that `exec`s a differently named binary, suspect a mismatch immediately
- If `kdotool search <AppName>` matches a terminal window, switch to class-based search terms and verify with KWin metadata
- If `desktopFile`, `resourceClass`, and `resourceName` all agree on a runtime ID, trust that over the visible launcher file name

## Example Outcome

Typical successful repair:

1. discover installed launcher `Foo.desktop`
2. inspect running window and find runtime identity `Foo.bin`
3. create `~/.local/bin/foo-desktop` that exports `CHROME_DESKTOP=Foo.bin.desktop`
4. create `~/.local/share/applications/Foo.bin.desktop` with `NoDisplay=true`
5. refresh desktop caches and relaunch

## Fallbacks

If the skill cannot identify the runtime ID through search:

- ask the user to focus the app briefly and use active-window inspection
- inspect process names and launcher scripts to infer likely runtime names
- check whether the app is really running under XWayland, Flatpak, or another packaging environment that needs a different fix
