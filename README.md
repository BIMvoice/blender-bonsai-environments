# Running several Bonsai versions side by side

Keep Bonsai **stable**, **unstable**, **BonsaiPR** and a **live development build** on the same machine, each in its own Blender environment, with no enabling or disabling when you switch between them.

You only install Blender once.

---

## The problem this solves

Blender extensions install their Python packages into a single shared folder per Blender profile. Two Bonsai builds in the same profile overwrite each other's files. That is why people end up enabling one add-on and disabling another every time they want to compare, and why "let me just check that against stable" turns into a five minute chore.

## The idea, in one sentence

Blender keeps everything it knows about you, meaning add-ons, preferences and Python packages, in one user folder. Point it at a different folder and you get a completely fresh Blender.

That is the whole trick. Everything below is applying it a few times.

## You install Blender once

This is the usual first question, so to be clear: **you do not need several copies of Blender.**

Think of it as one application with several separate user accounts. The program is installed once. Each account has its own add-ons and settings and cannot see the others'.

Two consequences worth knowing:

- When a new Blender version arrives you update **once** and every environment gets it.
- Every environment runs the identical Blender build, so any difference you see is genuinely the Bonsai variant, not the Blender version. That is exactly what you want when comparing stable against a test build.

## What actually goes in these folders

Not Blender. A Blender **profile**, which looks like this:

```
<environment folder>/config       preferences, keymaps, startup file
<environment folder>/extensions   add-ons, including Bonsai and its Python packages
<environment folder>/scripts      your own scripts
```

Expect roughly 600 MB per environment once Bonsai is installed.

## Which build should be your normal Blender?

Your default Blender, the one in Applications or the Start menu, is what opens when you double-click a `.blend` or `.ifc`, or open something a colleague sent you. **That happens by accident, so it should be a build that is safe to open real project work in.**

The rule worth following: **your most-used build that is safe for real work**, not simply your most-used build.

For most people that is stable or unstable. A live development build is a poor default, because a client file could open in half-finished code that might write malformed IFC without visibly failing. Development work is deliberate anyway, so it costs nothing to put it behind its own icon.

## Where to put the environment folders

Anywhere you like, but **avoid a folder that syncs to the cloud.**

On macOS, if you have iCloud "Desktop and Documents" sync enabled, putting these in `~/Documents` will silently start uploading gigabytes of Blender profiles. The same applies to OneDrive or Dropbox folders on Windows.

If you are unsure, use a plain folder in your home directory such as `~/BlenderEnvs` or `C:\BlenderEnvs`. The examples below use `Documents` for readability; substitute your own path.

---

# macOS

Tested with Blender 5.2 on macOS.

## 1. Make the folders

```bash
mkdir -p ~/Documents/"Blender Environments"/{stable,pr,dev}
```

Only make folders for the variants that are **not** your default Blender. If your normal Blender is already unstable, you do not need an `unstable` folder.

## 2. Set up each variant, one at a time

```bash
BLENDER_USER_RESOURCES=~/Documents/"Blender Environments"/stable \
  /Applications/Blender.app/Contents/MacOS/Blender
```

Blender opens looking brand new, because it is. Install that variant through **Edit > Preferences > Get Extensions**, then quit. Repeat with `pr` and `dev` in the path.

For `dev`, install **normal Bonsai** for now. That puts the compiled binary in place, which the live link section needs.

## 3. Make proper app launchers

A `.app` bundle can be pinned to the Dock, found in Spotlight, and opens no Terminal window.

**Make one only for the environments that are not your default Blender.** Your normal Blender already opens the build in its own profile, so a launcher for that one would be a second icon doing exactly the same thing. A silent duplicate is worse than an uneven-looking folder, because the point of this setup is always knowing which environment you are in.

So if your default Blender runs unstable, you want three launchers, not four.

Create `Bonsai Stable.app/Contents/MacOS/launch` containing:

```bash
#!/bin/bash
exec /usr/bin/open -n -a "/Applications/Blender.app" \
  --env "BLENDER_USER_RESOURCES=$HOME/Documents/Blender Environments/stable"
```

**Use `open` rather than running Blender's binary directly.** This matters for two reasons that are not obvious:

- Running the binary inside your wrapper gives you **two Dock icons**, the wrapper and Blender.
- More importantly, it makes macOS treat Blender as a **new, unknown application**, which is then denied access to protected folders such as Documents. The symptom is a `Repository Alert: Operation not permitted` error inside Blender. Using `open` keeps Blender's own identity and its existing permissions.

`Contents/Info.plist` needs at minimum a `CFBundleExecutable` of `launch`, plus `CFBundleName`, `CFBundleIdentifier` and `CFBundlePackageType` of `APPL`. Make `launch` executable with `chmod +x`.

## 4. Add a console launcher for logs

The apps hide Blender's output. When you want to see Python errors, tracebacks or your own `print()` calls, use a `.command` file that runs Blender **inside** Terminal:

```bash
#!/bin/bash
export BLENDER_USER_RESOURCES="$HOME/Documents/Blender Environments/dev"
"/Applications/Blender.app/Contents/MacOS/Blender" "$@"
echo "Blender exited with status $?"
read -p "Press return to close."
```

The trailing `read` keeps the window open so a crash message does not vanish with it. `chmod +x` this too.

For your default Blender, use the same file without the `export` line. That one **is** worth having even though it has no `.app`, because it does something the normal Blender icon cannot: it shows you the log.

Name all of them consistently. If three are called `Bonsai Stable`, `Bonsai PR` and `Bonsai Dev`, the fourth should be `Bonsai Unstable`, not `Blender Unstable`, even though it launches the default profile. From your side they are all Bonsai environments.

---

# Windows

**Not tested by the author.** The mechanism is identical and the environment variable is the same; the paths and launcher format differ. Corrections via issues or pull requests are welcome.

## 1. Make the folders

```
C:\BlenderEnvs\stable
C:\BlenderEnvs\pr
C:\BlenderEnvs\dev
```

Avoid OneDrive-synced locations.

## 2. Make a launcher for each

In Notepad, save as `Bonsai Stable.bat` with **All Files** as the type:

```bat
@echo off
set "BLENDER_USER_RESOURCES=C:\BlenderEnvs\stable"
start "" "C:\Program Files\Blender Foundation\Blender 5.2\blender.exe"
```

For a version that shows the log, drop the `start ""` and call `blender.exe` directly, then add `pause` at the end.

## 3. Install a Bonsai build in each

Launch each one and install its variant through **Edit > Preferences > Get Extensions**.

---

# Where to get each Bonsai build

**Stable.** Blender's own extensions platform. Edit > Preferences > Get Extensions, search Bonsai.

**Unstable.** Add this repository under Preferences > Get Extensions > Repositories:

```
https://raw.githubusercontent.com/IfcOpenShell/bonsai_unstable_repo/main/index.json
```

**BonsaiPR.** An unofficial build by falken10vdl bundling open pull requests, so you can test a fix before it is merged:

```
https://raw.githubusercontent.com/falken10vdl/bonsaiPR/refs/heads/main/index.json
```

Two references worth reading together:

- Written steps with screenshots: <https://github.com/falken10vdl/bonsaiPR#installation-with-automated-updates>
- Video walkthrough: <https://youtu.be/j5LAJSVNxmU>

Things worth knowing about BonsaiPR:

- Blender **checks** for new builds on startup and tells you when one exists, but it does **not** install them for you. You click Update.
- Its Python module is named **`bonsaiPR`**, not `bonsai`. That is why it can coexist with regular Bonsai, and it is worth remembering when you read a traceback and wonder why the paths say `bonsaiPR`.
- falken10vdl's own instructions begin by telling you to disable regular Bonsai first. **With this setup you do not need to**, because each build sits in its own profile. That step exists precisely because of the problem this guide solves.

## The trap this setup avoids

If you install BonsaiPR alongside regular Bonsai in the same profile, they share one `site-packages`. Disabling BonsaiPR then deletes the Python packages it installed there, including `ifcopenshell` and every other dependency, leaving regular Bonsai unable to start at all, with no interface and no obvious cause.

The recovery is to enable regular Bonsai again so Blender re-extracts its bundled packages. Separate profiles make it impossible.

---

# The live development link

Only needed if you want an environment running code straight from a git checkout.

## Use the project's own script

IfcOpenShell ships one, and it is better than doing this by hand:

```
src/bonsai/scripts/dev_environment.py
```

From its own documentation:

> Script links existing Bonsai installation to the provided IfcOpenShell repository.

It handles the platform differences and the compiled binaries, and has a `--skip-binaries` flag if you already have current ones. Install Bonsai normally in the target environment first, then run it.

## What it does, for the curious

Bonsai is mostly plain Python, but part of it is a **compiled binary** that only comes from a real install. So the pure Python packages are replaced with links to your checkout, while the binary is copied into the checkout.

Roughly, in `<profile>/extensions/.local/lib/python<X.Y>/site-packages`:

- rename `bonsai` and `ifcopenshell` aside, keeping them rather than deleting
- symlink both names to `<checkout>/src/bonsai/bonsai` and `<checkout>/src/ifcopenshell-python/ifcopenshell`
- copy `_ifcopenshell_wrapper*` and `ifcopenshell_wrapper.py` from the kept copies into the checkout

On Windows the compiled file is `.pyd` rather than `.so`, and directory links need Developer Mode or an Administrator prompt.

## Two things that will confuse you later

**The extension stays registered against whichever repository you installed from.** Blender uses that entry to enable the add-on, while Python imports through the symlink. It works, but **do not click Update on Bonsai in that environment**: it will download a normal build over the top and silently replace your live link. Your edits will simply stop having any effect, with no error.

**A source checkout reports its version as `0.0.0`.** That is normal. Bonsai's debug panel will show the real commit hash and branch, which is the better thing to check.

---

# Confirm which environment you are in

The windows look identical. Before reporting a bug, check what you are actually running. In Blender's Python console:

```python
import bonsai; print(bonsai.__file__)
```

For BonsaiPR, `import bonsaiPR` instead.

The folder in the printed path tells you which environment you are in.

---

# Things that catch people out

**The Python version is in that path.** `python3.13` is specific to Blender 5.2. A different Blender uses a different number, and the compiled binary is built for one exact Python version.

**Your existing Blender profile is untouched** by any of this, unless you deliberately link it.

**Each environment downloads its own dependencies.** Around 600 MB each.

**Check a test build supports your Blender version** before assuming the setup is broken.

---

# Why not just install several Blender versions?

You can, and it needs no setup at all, because Blender already separates profiles by version.

There are three catches, and the first is the one people feel most.

**You get frozen on old Blender versions.** To keep an old Bonsai around you have to keep the Blender that came with it, so you lose everything that improved in Blender itself in the meantime. Tools, performance, interface changes, none of which have anything to do with BIM. Your testing setup ends up holding you back from the thing you actually model in.

**You change two variables at once.** When something breaks you cannot tell whether it was the Bonsai build or the Blender version, because both differ. That is precisely the question you were trying to answer.

**You maintain several Blender installs forever**, updating each one separately.

One Blender and several profiles avoids all three. Every version runs on the same current Blender, so the only thing that differs is Bonsai.

---

# Setting this up with an AI assistant

If you use Claude Code, Codex, Gemini CLI or a similar tool, point it at [AGENTS.md](AGENTS.md). It contains step by step instructions written for agents, including the two steps that cannot be automated and must be handed back to you.

---

# Credits and corrections

Written up from a working macOS setup. The Windows half is untested by the author, so corrections are genuinely welcome. Issues and pull requests are open.

Bonsai and IfcOpenShell are separate projects. This guide is not affiliated with them; it just describes a way of running them.
