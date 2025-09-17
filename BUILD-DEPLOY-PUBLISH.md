# Build, Deploy, and Publish (lib-df-buttons and df-curvy-walls)

This guide shows how to build, deploy locally to FoundryVTT, and publish releases for the modules:
- `lib-df-buttons`
- `df-curvy-walls`

It uses the DOIT build scripts embedded in this repository and assumes you run commands inside WSL (Ubuntu) on Windows. Adjust paths to your environment if needed.

---

## 1) Prerequisites (WSL)

Run these inside your Ubuntu WSL terminal.

- Install required tools
  - jq, zip, entr, Python 3, Node.js + npm

```bash
sudo apt update
sudo apt install -y jq zip entr python3
sudo apt install -y nodejs npm
```

- Optional: install Node via nvm for a newer version
```bash
# Optional
curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
# Restart shell, then:
nvm install --lts
```

- Install the DOIT runner (CLI)
  - Download from: https://github.com/flamewave000/doit/releases/latest
  - Place the binary into `/usr/local/bin` and make it executable
```bash
sudo install -m 0755 ~/Downloads/doit /usr/local/bin/doit
```

- Clone or open the repo in WSL and install project dev deps (sass, etc.)
```bash
cd /mnt/a/Projects/GIT/dragonflagon-fvtt
#doit help check
doit init -g
```

---

## 2) Configure Foundry paths

Given your FoundryVTT folder on Windows is `A:\FoundryVTT`, use the corresponding WSL path `/mnt/a/FoundryVTT`.

Pick ONE of the following based on your setup:

```bash
# Case 1: Foundry install and Data are both at A:\FoundryVTT (contains a Data/ folder)
doit env /mnt/a/FoundryVTT /mnt/a/FoundryVTT

# Case 2: Foundry install is A:\FoundryVTT and data is A:\FoundryVTT\Data
doit env /mnt/a/FoundryVTT /mnt/a/FoundryVTT/Data
```

The deploy script auto-detects whether to use `Data/modules` or `modules` under the configured data root.

---

## 3) Build and deploy locally

The build scripts:
- compile `.scss` to `.css` and remove the `.scss` in the deployed/packaged output
- expand `{{sources}}` and `{{css}}` in `module.json`
- copy the module into your Foundry Data `modules` folder for testing

### 3.1 lib-df-buttons

```bash
doit tgt lib-df-buttons
# one-time per target
doit init
# deploy into Foundry's modules folder
doit dev
```

Enable “Library: DF Module Buttons” in Foundry.

Optional: watch and auto-deploy on changes
```bash
doit watch
```

### 3.2 df-curvy-walls

`df-curvy-walls` requires `lib-wrapper` and `lib-df-buttons`. Ensure both exist and are enabled in Foundry.

```bash
doit tgt df-curvy-walls
# one-time per target
doit init
# deploy into Foundry's modules folder
doit dev
```

Optional watch:
```bash
doit watch
```

---

## 4) Package for release

For the currently selected target module:

```bash
doit pack
```

Produces:
- `release/<module>_<version>/module.json`
- `release/<module>_<version>/<module>.zip`

> Note: If you see `sh: zip: not found`, install zip inside WSL and re-run:
> ```bash
> sudo apt update && sudo apt install -y zip
> which zip   # should print /usr/bin/zip
> ```
> If you trigger the pack from PowerShell, run it via WSL:
> ```powershell
> wsl -e bash -lc "cd /mnt/a/Projects/GIT/dragonflagon-fvtt && doit pack"
> ```

Repeat by switching targets to package the other module.

---

## 5) Publish for users (GitHub Releases)

Foundry installs use a manifest URL pointing to your module’s `module.json`, which must include a `download` URL for the `.zip`.

### 5.1 Tag the release

Tags are per module and match the version in that module’s `module.json`.

```bash
# Ensure correct target first
doit tag
# Creates/pushes a tag like: lib-df-buttons_2.0.3 or df-curvy-walls_4.0.0
```

### 5.2 Create a GitHub Release

- On GitHub → Releases → “Draft a new release”
- Select the tag created above
- Upload two assets from `release/<module>_<version>/`:
  - `module.json`
  - `<module>.zip`
- Publish the release

### 5.3 Manifest and download URLs

Before packaging/uploading, update each module’s `module.json` to point to your GitHub release URLs, for example:

- manifest (stable):
  - `https://github.com/heitorpfigueira/dragonflagon-fvtt/releases/latest/download/module.json`
- download (per tag):
  - `https://github.com/heitorpfigueira/dragonflagon-fvtt/releases/download/<TAG>/<module>.zip`

Examples:
- lib-df-buttons
  - manifest: `https://github.com/heitorpfigueira/dragonflagon-fvtt/releases/latest/download/module.json`
  - download: `https://github.com/heitorpfigueira/dragonflagon-fvtt/releases/download/lib-df-buttons_<version>/lib-df-buttons.zip`
- df-curvy-walls
  - manifest: `https://github.com/heitorpfigueira/dragonflagon-fvtt/releases/latest/download/module.json`
  - download: `https://github.com/heitorpfigueira/dragonflagon-fvtt/releases/download/df-curvy-walls_<version>/df-curvy-walls.zip`

After you publish the release, share the manifest URL. Users can paste it into Foundry’s “Install Module” dialog.

> Tip: You can also host manifests/zips on GitHub Pages or any static host if you prefer.

---

## 6) Optional helpers

- Launch (Node distribution only):
```bash
doit launch
```

- Symlink Foundry public scripts for better editor tooling:
```bash
doit getref
```

---

## 7) Troubleshooting

- WSL paths:
  - Use `/mnt/a/FoundryVTT` instead of `A:\FoundryVTT`.
  - The deploy step detects `Data/` under the data root and targets `Data/modules` or `modules` accordingly.

- Missing tools:
  - `sh: zip: not found` when running `doit pack`:
    - Install `zip` in WSL: `sudo apt update && sudo apt install -y zip`
    - Ensure you execute the command in WSL, or wrap from PowerShell:
      - `wsl -e bash -lc "cd /mnt/a/Projects/GIT/dragonflagon-fvtt && doit pack"`
  - If `doit` not found → install and ensure it’s on PATH.
  - If `npx sass` fails → ensure `doit init -g` ran and Node/npm are installed in WSL.

- Dependencies:
  - `df-curvy-walls` requires `lib-wrapper` and `lib-df-buttons`.

- SCSS:
  - Compiled to CSS and `.scss` files are removed in the deployed/packaged folder.

- Cleaning:
  - `doit clean` → cleans target symlinks (`common/`).
  - `doit clean -g` → removes global build artifacts (`node_modules`, `release/`, etc.).

---

## Quick reference

```bash
# Initial setup
cd /mnt/a/Projects/GIT/dragonflagon-fvtt
doit init -g
doit env /mnt/a/FoundryVTT /mnt/a/FoundryVTT   # or /mnt/a/FoundryVTT/Data

# lib-df-buttons
doit tgt lib-df-buttons
doit init
doit dev

# df-curvy-walls
doit tgt df-curvy-walls
doit init
doit dev

# Package current target
doit pack

# Tag current target
doit tag
```
