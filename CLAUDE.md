# Claude Code

See [AGENTS.md](AGENTS.md) for the full instructions on setting this up on a user's machine.

The two things worth knowing before you start:

1. **Installing a Bonsai extension requires the GUI** and cannot be automated. Set up everything else, then hand those steps back to the user explicitly. Do not report the task complete while any environment is still empty.

2. **Do not hardcode the Python version** in profile paths. Ask Blender:

   ```bash
   "<blender>" --background --python-expr "import sys;print(sys.version_info[:2])"
   ```
