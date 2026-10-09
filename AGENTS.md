# minecraft-addon-workflows

**Not an add-on — this is build infrastructure.** It holds the
`minecraft-pack-build` script and the reusable GitHub Actions workflow that
every other project consumes.

- `minecraft-pack-build` — local build script (bash, 858 lines).
- `.github/workflows/build-and-release.yml` — reusable workflow.
- `templates/` — **empty**, and the README never mentions it.
- `BP/` — empty directory structure, no `manifest.json`.

There is no `.github/templates/` here; that path belongs to the *consuming*
projects.

## Connecting a project (README, 3 steps)

1. Create `.github/templates/release.md` in the project with placeholders
   `{{ version }}`, `{{ bp_files }}`, `{{ rp_files }}`, `{{ total_blocks }}`,
   `{{ total_items }}`, `{{ total_models }}`, `{{ locales_list }}`, `{{ commits }}`.
2. Create `.github/workflows/build.yml` calling
   `flower7c3/minecraft-addon-github-workflows/.github/workflows/build-and-release.yml@main`
   with `secrets: inherit`.
3. Required in the project: `BP/manifest.json`, `RP/manifest.json`,
   `version.json`, `.github/templates/release.md`, an executable
   `minecraft-pack-build`.

## `minecraft-pack-build`

Flags: `-i DIR` (source, required), `-o DIR` (output, default `<input>dist/`),
`-f/--force` (create output dir), `-m DIR` (Minecraft dir), `-a/--mcaddon`,
`-p/--mcpack`, `-l/--local-installation`, `-d/--debug-installation`,
`-b/--no-bump`, `-v/--no-version`, `-t/--no-time`.

Requires `jq`, `zip`, `realpath`, `bc`, `tput`, `dd`, `wc` (bash ≥ 4).

`bump_versions()` increments **only the patch** (`.[2] += 1`) in `version.json`,
both manifests, and propagates the version into `dependencies` — matching own
packs **by UUID**, not by position, so foreign dependencies are left alone.

## Risks — read this before running `-l` or `-d`

- 🔴 **`minecraft-pack-build:679-680`**, repeated in `debug_locally`:
  ```bash
  rm -rf "${MOJANG_DIR}minecraftWorlds/*/behavior_packs/"
  rm -rf "${MOJANG_DIR}minecraftWorlds/*/resource_packs/"
  ```
  Runs on **every** `-l` and `-d`. It deletes packs from **all worlds**, not just
  this project's. If `-m` points at the wrong directory, it destroys data.
- `confirm_or_exit()` and `prompt_input()` are defined but **never called** —
  every `rm -rf` and manifest overwrite happens without asking.
- `check_dependencies()` is likewise never called, so a missing `jq` produces a
  confusing failure instead of a readable message.
- `PACKNAME` is derived from the directory name via `sed`/`awk`
  (`personal-music-compilation` → `Personal Music Compilation`, with spaces).
  If that normalisation diverges from the name used when zipping, `find -name`
  won't match old archives and `dist/` accumulates stale builds.
- `-d` writes nothing to `OUTPUT_DIR`, yet validation requires it to exist —
  the Makefile passes `--force` only to create `dist/`.
- `bc -z` doesn't exist on macOS; handled by a `|| echo` fallback.
- `export TERM=xterm-256color` is set **twice** and leaks into the calling
  shell if the script is `source`d.
- `-d/--debug-installation` is missing from `--help`.
- The literal `"pack.name"` placeholder in manifests comes from **Bridge's
  `simpleRewrite` plugin, not from this script** — it injects nothing itself.

## Used by other projects

`projects/Makefile` copies this script to `~/.local/bin` (`make sync-bin`) and
builds all 11 projects from its `PROJECTS` list. `install.sh` duplicates that
list in a second place — **update both when adding a project.**