# Instructions for AI coding agents

If a user asks you to set up multiple Bonsai environments using this guide, follow this. It is written for Claude Code, Codex, Gemini CLI and similar tools.

Read the whole file before running anything. Two steps genuinely cannot be automated, and knowing which they are up front will save the user a confusing half-finished setup.

---

## What you are building

One Blender install, several isolated user profiles. Blender's `BLENDER_USER_RESOURCES` environment variable replaces the entire user directory, so each profile gets its own extensions, preferences and Python packages. That isolation is the whole point: without it, two Bonsai builds overwrite each other's files in a shared `site-packages`.

You are **not** installing Blender several times. If you find yourself downloading Blender more than once, you have misread the task.

## What you cannot do, and must hand back to the user

**1. Installing an extension.** Bonsai is installed through Blender's GUI (Edit > Preferences > Get Extensions). There is no reliable headless equivalent across Bonsai's release channels. Do not attempt to fake it by copying files into `site-packages`: the extension also needs registry and preference entries, and a hand-copied package will not load.

**2. Confirming it works in the GUI.** You can verify imports headlessly. You cannot confirm the add-on panels appear.

Set up everything else, then tell the user plainly which steps are theirs. Do not report the task complete while those remain.

---

## Step 1: discover, do not assume

Three things vary per machine. Detect all three rather than hardcoding.

**The Blender executable.**

- macOS: `/Applications/Blender.app/Contents/MacOS/Blender`
- Windows: `C:\Program Files\Blender Foundation\Blender <version>\blender.exe`
- Linux: often `/usr/bin/blender` or a downloaded folder

Confirm it runs:

```bash
"<blender>" --version
```

**The Blender version**, from that output. It determines the profile layout.

**The Python version Blender embeds.** This is the one people get wrong. Ask Blender itself:

```bash
"<blender>" --background --python-expr "import sys;print('PY', sys.version_info[:2])"
```

Blender 5.2 uses Python 3.13, but do not rely on that. The path you will need later contains this number, and the compiled binary in step 5 is built for one exact Python version.

## Step 2: create the profile folders

Somewhere visible to the user. `~/Documents/Blender Environments/` is a good default. Avoid developer-looking paths; many users of this guide are not developers.

Create one folder per variant the user asked for, typically `stable`, `unstable`, `pr`, `dev`.

## Step 3: verify isolation before going further

Cheap, and it catches a wrong Blender path immediately:

```bash
BLENDER_USER_RESOURCES="<folder>/stable" "<blender>" --background \
  --python-expr "import bpy;print('PROFILE', bpy.utils.resource_path('USER'))"
```

The printed path must be the folder you passed. If it prints the user's normal Blender directory, the variable is not taking effect and nothing after this will work.

## Step 4: create launchers

**macOS.** Prefer a real `.app` bundle over a `.command` file. Apps can be pinned to the Dock, appear in Spotlight, and open no Terminal window. Minimum structure:

```
Bonsai Stable.app/Contents/Info.plist
Bonsai Stable.app/Contents/MacOS/launch     (executable)
```

`launch` is a shell script that exports `BLENDER_USER_RESOURCES` and `exec`s Blender. `Info.plist` needs `CFBundleExecutable` set to `launch`, plus a name and identifier. Mark the launcher executable.

Warn the user that macOS may block an unsigned app on first run, and that right-click > Open clears it permanently.

**Windows.** A `.bat` per environment:

```bat
@echo off
set "BLENDER_USER_RESOURCES=%USERPROFILE%\Documents\Blender Environments\stable"
start "" "C:\Program Files\Blender Foundation\Blender 5.2\blender.exe"
```

**Linux.** A shell script, or a `.desktop` file if the user wants menu entries.

## Step 5: the live development link, only if asked

Only relevant if the user wants an environment running a git checkout of IfcOpenShell.

Bonsai is mostly plain Python, but part of it is a **compiled binary** that only comes from a real install. So the order matters:

1. The user installs Bonsai normally into the `dev` environment, through the GUI. This provides the binary.
2. You then replace only the pure Python packages with links to the checkout.

In `<dev profile>/extensions/.local/lib/python<X.Y>/site-packages`:

- Rename `bonsai` and `ifcopenshell` to `bonsai.stock` and `ifcopenshell.stock`. **Rename, never delete.** The user needs a way back, and the stock copy is where the compiled binary comes from.
- Symlink both names to `<checkout>/src/bonsai/bonsai` and `<checkout>/src/ifcopenshell-python/ifcopenshell`.
- Copy the compiled wrapper from the stock copy into the checkout, since a source tree has none. It is `_ifcopenshell_wrapper*.so` on macOS and Linux, `*.pyd` on Windows, plus `ifcopenshell_wrapper.py`.

On Windows, directory symlinks need Developer Mode enabled or an Administrator prompt. If neither is available, say so rather than silently falling back to copying, which would break the "live" part entirely.

**Check the checkout's branch is compatible with the installed build.** If the binary came from one version of the code and the checkout has moved to an incompatible one, imports fail. As one real example, IfcOpenShell's `v0.9.0` rewrote the C++/Python boundary, so a `v0.8` binary cannot run `v0.9` Python source.

## Step 6: verify what you can

For each environment:

```bash
BLENDER_USER_RESOURCES="<folder>/<env>" "<blender>" --background --python-expr "
try:
    import bonsai, ifcopenshell, os
    print('OK', os.path.realpath(bonsai.__file__))
except ImportError:
    print('NONE')
"
```

Expected before the user installs anything: `NONE` everywhere, except any environment you linked to a checkout.

Expected afterwards: a path per environment, and **every path different**. Two environments reporting the same source means isolation has failed, and you should investigate rather than report success.

A source checkout reports `ifcopenshell.version` as `0.0.0`, which is normal and not a fault.

## Step 7: report honestly

Tell the user:

- which environments exist and where the launchers are
- exactly which steps remain theirs, namely installing a Bonsai variant into each empty environment through the GUI
- that you verified imports headlessly but not the GUI
- roughly how much disk each environment will use once populated, a few hundred MB

Do not describe the setup as finished while environments remain empty.

---

## Things that will trip you up

**Do not hardcode the Python version** in any path. Ask Blender.

**Do not copy an extension between profiles** as a shortcut for installing it, unless you copy the whole profile including its config. Extensions are registered in preferences, not just present on disk.

**Do not touch the user's existing Blender profile** without asking. It holds their real work setup. Everything here should live in new folders.

**Preserve symlinks when copying a profile.** Use `cp -a` on macOS and Linux, not `cp -R`, or you will silently copy the entire checkout into the profile and the live link will be gone while still appearing to work.

**Verify rather than assume, and say which is which.** If you did not run it, do not report it as working.
