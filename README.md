# Running several Bonsai versions side by side

Keep Bonsai **stable**, **unstable**, **BonsaiPR** and a **live development build** on the same machine, each in its own Blender environment, with no enabling or disabling when you switch between them.

You only install Blender once.

---

## The problem this solves

Blender extensions install their Python packages into a single shared folder per Blender profile. Two Bonsai builds in the same profile overwrite each other's files. That is why people end up enabling one add-on and disabling another every time they want to compare, and why "let me just check that against stable" turns into a five minute chore.

## The idea, in one sentence

Blender keeps everything it knows about you, meaning add-ons, preferences and Python packages, in one user folder. Point it at a different folder and you get a completely fresh Blender.

That is the whole trick. Everything below is applying it four times.

## You install Blender once

This is the usual first question, so to be clear: **you do not need four copies of Blender.**

Think of it as one application with four separate user accounts. The program is installed once. Each account has its own add-ons and settings and cannot see the others'.

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

Expect a few hundred MB per environment once Bonsai is installed. That is the cost of them being genuinely independent, and still far less than several Blender installs.

---

# macOS

Tested on macOS with Blender 5.2.

## 1. Make the folders

Open Terminal and run:

```bash
mkdir -p ~/Documents/"Blender Environments"/{stable,unstable,pr,dev}
```

## 2. Set up each variant, one at a time

Finish each one completely before starting the next.

**Stable:**

```bash
BLENDER_USER_RESOURCES=~/Documents/"Blender Environments"/stable \
  /Applications/Blender.app/Contents/MacOS/Blender
```

Blender opens looking brand new, because it is. Go to **Edit > Preferences > Get Extensions**, search for Bonsai, install it. Quit Blender.

**Unstable:** same command with `unstable` in the path. Add the Bonsai unstable repository in Preferences, install from it, quit.

**BonsaiPR:** same command with `pr`. Install the BonsaiPR build, either from its repository or by dragging in the extension file. Quit.

**Dev:** same command with `dev`. Install **normal Bonsai** here for now. This is deliberate: it puts the compiled binary in place, which the live link section needs. Quit.

## 3. Make them double-clickable

Open **TextEdit**, choose **Format > Make Plain Text**, and type:

```bash
#!/bin/bash
BLENDER_USER_RESOURCES=~/Documents/"Blender Environments"/stable \
  /Applications/Blender.app/Contents/MacOS/Blender
```

Save it into your `Blender Environments` folder as `Bonsai Stable.command`. Repeat for the other three, changing both the folder name and the file name.

| File name | Folder |
|---|---|
| `Bonsai Stable.command` | `stable` |
| `Bonsai Unstable.command` | `unstable` |
| `Bonsai PR.command` | `pr` |
| `Bonsai Dev.command` | `dev` |

Then once, in Terminal:

```bash
chmod +x ~/Documents/"Blender Environments"/*.command
```

Now they launch from Finder or the Dock. That `chmod` is the only Terminal step you cannot avoid, and it is once, forever.

---

# Windows

**Not tested by the author.** The mechanism is identical and the environment variable is the same; the paths and the launcher format differ. Corrections via issues or pull requests are welcome.

Adjust `5.2` and the install path to match your Blender.

## 1. Make the folders

In File Explorer, create:

```
C:\Users\<you>\Documents\Blender Environments\stable
C:\Users\<you>\Documents\Blender Environments\unstable
C:\Users\<you>\Documents\Blender Environments\pr
C:\Users\<you>\Documents\Blender Environments\dev
```

## 2. Make a launcher for each

Open **Notepad** and type, on one line each:

```bat
@echo off
set "BLENDER_USER_RESOURCES=%USERPROFILE%\Documents\Blender Environments\stable"
start "" "C:\Program Files\Blender Foundation\Blender 5.2\blender.exe"
```

Save as `Bonsai Stable.bat`, choosing **All Files** as the type so Notepad does not add `.txt`. Repeat for the other three, changing the folder name.

Double-click the file to launch that environment. Right-click and **Pin to Start** if you want them handy.

## 3. Install a Bonsai build in each

Launch each one and install its Bonsai variant through **Edit > Preferences > Get Extensions**, exactly as on macOS. What you install goes only into that environment.

---

# The live development link

Only needed if you want an environment running code straight from a git checkout. Skip this if you just want the released builds.

Bonsai is mostly plain Python, but part of it is a **compiled binary** that only comes from a real install. So you install Bonsai normally first, then replace only the Python with links to your checkout.

## macOS

```bash
cd ~/Documents/"Blender Environments"/dev/extensions/.local/lib/python3.13/site-packages
```

Keep the originals so this is reversible:

```bash
mv bonsai bonsai.stock
mv ifcopenshell ifcopenshell.stock
```

Point the names at your checkout, adjusting the paths to your own:

```bash
ln -s /path/to/IfcOpenShell/src/bonsai/bonsai bonsai
ln -s /path/to/IfcOpenShell/src/ifcopenshell-python/ifcopenshell ifcopenshell
```

Copy the compiled binary into the checkout, since a source tree does not contain one:

```bash
cp ifcopenshell.stock/_ifcopenshell_wrapper*.so \
   /path/to/IfcOpenShell/src/ifcopenshell-python/ifcopenshell/
cp ifcopenshell.stock/ifcopenshell_wrapper.py \
   /path/to/IfcOpenShell/src/ifcopenshell-python/ifcopenshell/
```

To undo: delete the two links and rename the `.stock` folders back.

## Windows

Same idea, but Windows needs either **Developer Mode** enabled or an **Administrator** command prompt to create directory links.

```bat
cd "%USERPROFILE%\Documents\Blender Environments\dev\extensions\.local\lib\python3.13\site-packages"

ren bonsai bonsai.stock
ren ifcopenshell ifcopenshell.stock

mklink /D bonsai "C:\path\to\IfcOpenShell\src\bonsai\bonsai"
mklink /D ifcopenshell "C:\path\to\IfcOpenShell\src\ifcopenshell-python\ifcopenshell"

copy ifcopenshell.stock\_ifcopenshell_wrapper*.pyd "C:\path\to\IfcOpenShell\src\ifcopenshell-python\ifcopenshell\"
copy ifcopenshell.stock\ifcopenshell_wrapper.py "C:\path\to\IfcOpenShell\src\ifcopenshell-python\ifcopenshell\"
```

Note the compiled file is `.pyd` on Windows rather than `.so`.

---

# Confirm which environment you are in

The windows look identical. Before reporting a bug, check which build you are actually running. Open Blender's Python console and run:

```python
import bonsai; print(bonsai.__file__)
```

The folder name in the printed path tells you immediately whether you are in `stable`, `unstable`, `pr` or `dev`.

---

# Things that catch people out

**The Python version is in that path.** `python3.13` is specific to Blender 5.2. A different Blender uses a different number. This matters most for the compiled binary, which is built for one exact Python version: copying a `cpython-313` file into a Blender running Python 3.11 will not import.

**Your existing Blender profile is untouched.** Everything here lives in the new folders. Launching Blender normally still behaves exactly as it did before, with whatever you already had installed.

**The dev environment follows whichever branch your checkout is on.** If the compiled binary came from one version of the code and your checkout has moved to an incompatible one, imports will fail. Keep the checkout on a branch matching your installed build.

**Check that a test build supports your Blender version** before assuming it will run. Not every Bonsai variant targets every Blender release.

---

# Why not just install several Blender versions?

You can, and it needs no setup at all, because Blender already separates profiles by version.

The catch is that you are then changing two things at once. If something breaks, you cannot tell whether it was the Bonsai build or the Blender version. You would also be maintaining several Blender installs forever.

One Blender and several profiles keeps the comparison honest.

---

# Credits and corrections

Written up from a working macOS setup. The Windows half is untested by the author, so corrections are genuinely welcome. Issues and pull requests are open.

Bonsai and IfcOpenShell are separate projects. This guide is not affiliated with them; it just describes a way of running them.
