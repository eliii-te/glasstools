# glasstools 🛠️

The **closed system tooling of GlassyOS** — a Linux distribution built on top
of Arch Linux, designed to be beautiful, safe and low-maintenance.

> 🔒 **Made exclusively for GlassyOS.** Everything in this repository is a
> **closed, tightly integrated part of the GlassyOS ecosystem** — built
> specifically for GlassyOS, around its architecture, its lifecycle and its
> philosophy. These are **not** generic public utilities: they are private
> systems of the distribution, engineered to work together as one unit. The
> source is public to show the craft behind GlassyOS — but every tool is
> designed for GlassyOS first and only.

> 🗓️ **GlassyOS will be released next year.** Until then, this repository is
> the living workshop of everything the distribution ships with — update
> systems, security scanners, onboarding, boot theming and the GlassVibe
> media apps. All developed here, all exclusive to GlassyOS.

Everything is **self-contained, hand-written and home-grown**. No frameworks,
no bloat — plain Python 3 and Bash, the way system tools should be. Two closed
areas live in this repository:

| Area | Contents |
|---|---|
| [`bin/`](#system-tools-bin) | 13 system tools — update, rollback, security, health, install & more |
| [`apps/`](#glassvibe-apps-apps) | **GlassVibe** media apps — full user-facing tools with own setup installer |

---

## System tools (`bin/`)

The everyday system layer of GlassyOS — exclusive tooling that ships with the
distribution. Install by copying `bin/glassy-*` into your PATH:

```bash
install -m755 bin/glassy-* ~/.local/bin/
```

### 🔄 glassy-update — the GlassyOS smart updater (Python TUI)

**What it is.** The heart of the GlassyOS update experience. An interactive,
full-screen terminal UI with an animated splash, an arrow-key menu and real
update actions — every action is wrapped in a **rolling snapshot backup**, so
the system can always be rewound afterwards.

**How it works — the UI layer.** The tool talks to the terminal in raw mode
(`termios` + `tty`, its own `RawMode` context manager) and reads arrow keys
directly instead of using a TUI framework. It renders an animated splash, a
chromed frame (`render_chrome`) with title/subtitle and a footer, and paints
the main menu from scratch on every keypress. Everything degrades gracefully
to plain output when no TTY is available.

**How it works — snapshots.** Before any update, `make_snapshot()` creates a
timestamped folder (`YYYYMMDD-HHMMSS-<reason>`) under
`/var/lib/glassyos/backups` (or `~/.local/share/glassy/backups` if not
writable — the tool probes with a touch file and falls back automatically).
A snapshot contains:
- `pkglist-explicit.txt` — `pacman -Qqe`, the packages you explicitly installed
- `pkglist-aur.txt` — `pacman -Qqm`, packages from the AUR
- `pkglist-all.txt` — `pacman -Qq`, every installed package
- `mirrorlist` — a copy of `/etc/pacman.d/mirrorlist`
- `glassyos-release` — the active GlassyOS version file

Only the **last 10 snapshots** are kept; older ones are removed automatically.

**How it works — the actions.** The main menu offers:
- **Update GlassyOS** — reads `release_manifest_url` from the config
  (`/etc/glassyos/updater.conf` or `~/.config/glassy/updater.conf`, keys
  `release_manifest_url` + `channel`), downloads the release manifest (JSON),
  compares the announced version against the installed one
  (`/etc/glassyos-release`) and — if newer — downloads the release tarball and
  applies it with `tar --use-compress-program=unzstd -xf ... -C /`.
- **Update mirrors** — re-rates the fastest HTTPS mirrors via `rate-mirrors`
  (latest 20, protocol https, sorted by rate) and saves a fresh mirrorlist.
- **Update repos** — `sudo pacman -Sy --needed --noconfirm` on the configured
  repo groups (the GlassyOS internal repos).
- **Update apps** — updates user-installed applications.
- **Update everything** — the full cascade: mirrors → repos → apps → system.

**Admin/publish console.** Hidden behind a **secret trigger sequence** typed in
the main menu, plus the user's **login PIN** and an **admin password** (both
stored as salted SHA-256 hashes). This gates publish/release actions for the
GlassyOS maintainer — a self-contained release workflow inside the updater.

---

### ⏪ glassy-downgrade — roll the system back (Python TUI)

**What it is.** The power-user companion to `glassy-update`: an interactive
TUI to rewind parts of GlassyOS / Arch to an earlier state. Uses a red /
“danger” color palette to underline that this is a power tool — rewinding a
rolling-release system can break package signatures, leave dangling
dependencies and confuse pacman's database. It warns loudly, but does not stop
you — that's the point.

**How it works.** Three independent rollback paths:
1. **System downgrade by date** — lists the last 24 months of
   `archive.archlinux.org/repos`, lets you pick a date and then runs
   `sudo pacman -Syyuu --noconfirm` against the archive snapshot, rolling the
   whole system back to that point in time.
2. **GlassyOS downgrade** — reads locally available GlassyOS releases
   (snapshot folders / release dir) and applies an older one, again via tar
   extraction to `/`.
3. **Single package downgrade** — for a specific package it first searches the
   local pacman cache (`/var/cache/pacman/pkg`), then queries the Arch Linux
   archive for older versions, and hands the selection to **fzf**
   (rounded border, custom prompt) before installing with
   `sudo pacman -U`.

Before anything destructive it creates the same snapshot backups as
`glassy-update`, so the pre-rollback state is always recoverable.

---

### 👋 glassy-onboard — the first-boot welcome tour (GTK4/libadwaita)

**What it is.** A fullscreen GUI welcome tour that runs automatically on the
very first GlassyOS session (wired in via Hyprland `exec-once`, and gated by a
`~/.config/glassy/onboard-done` marker so it only appears once).

**How it works.** It's a GTK4 + libadwaita app with **smooth carousel pages**.
A shared `page_shell()` scaffold builds every page from three parts: a small
*eyebrow* tag on top, a big *title*, and a body block — all centered. The
pages include:
- an **INTRODUCTION** page (“What is glassyOS”) with a Cairo-animated logo,
- **key-chip rows** — keyboard-shortcut chips rendered as styled GTK labels,
  connected with little “+” separators,
- a **Waybar mock** — a fake status bar drawn with CSS classes to preview the
  desktop look,
- **per-tool cards** — one card per `glassy-*` command, explaining the closed
  tool ecosystem („Every system task has a glassy-* command“).

Styling is done with inline CSS (rounded cards, soft backgrounds, custom
popup chrome) — no image assets, everything drawn live.

---

### 🛡️ glassy-checkup — malware & integrity scanner

**What it is.** The on-demand security scanner: ClamAV + rkhunter + own
heuristic checks, with a built-in quarantine system. Can be run quickly or as
a full system sweep.

**How it works.** It dispatches on a **mode** — `--quick`, `--full`,
`--update`, or a custom path argument:
- Each mode defines its own scan profile: which directories ClamAV recurses
  over, where to hunt for suspicious SUID binaries, where to look for
  executables in writable temp locations, whether to scan `$HOME` for hidden
  executables, and whether rkhunter + a full `paccheck` package-integrity run
  are included.
- `--quick` scans `~/Downloads`, `~/.local/bin` and `/tmp` with the heuristic
  hunts, skipping the heavy scanners.
- `--full` runs ClamAV system-wide **and** rkhunter **and** a package
  integrity check.
- `--update` refreshes virus signatures (`freshclam`) and rkhunter's database.
- Missing tools (clamav, rkhunter) are offered for auto-install; the ClamAV
  signature service is enabled and an initial `freshclam` run is triggered.

**Quarantine.** Threats are moved into a managed quarantine area. The system
can be verified end-to-end with `--test`, which drops the harmless, universally
recognized **EICAR test string** into `~/Downloads`, watches the scanner catch
it and quarantines it. `--quarantine` lists the quarantine, `--restore <ID>`
puts files back, `--purge-quarantine` deletes everything.

---

### 🕵️ glassy-secure — real-time download scanner (daemon)

**What it is.** The always-on guardian: a small daemon that watches
`~/Downloads` **in real time** and scans every new file with ClamAV the moment
it appears. Runs as a user systemd service (`glassy-secure.service`).

**How it works.** It uses **inotify** to watch the download folder. Partially
written files (`.part`, `.crdownload`, `.tmp`) are skipped so the scan only
happens once a download has finished. New files are handed to `clamscan`;
detected threats are **auto-quarantined** (moved to
`~/.local/share/glassy/quarantine`) and the user gets an alert. The tool ships
as a service unit and is one of the services that `glassy-doctor` /
`glassy-fix` probe and restart if it dies.

---

### 🩺 glassy-doctor — system health check + auto-fix

**What it is.** The diagnostic: probes the critical subsystems of a GlassyOS
session, tries to repair what's broken, and prints a clear summary.

**How it works.** It checks each subsystem in turn — **Bluetooth, Wi-Fi,
PipeWire, Waybar, Hyprland, autostart entries and polkit** — and reports per
check with ✓ (ok), ⚒ (fixed) or ✗ (failed). When something is down it tries to
start/repair it automatically (unless `--no-fix` is given). `-v/--verbose`
adds detail, and everything is logged with a `[doctor]` prefix.

---

### 🔧 glassy-fix — automatic self-repair

**What it is.** The automatic repair pass — the tool that actually *fixes*
what `glassy-doctor` found, aimed at unattended/scheduled use.

**How it works.** `glassy-fix` walks over the known GlassyOS **user services**
(`elephant`, `walker`, `glassy-secure`) plus desktop components: for every
service it reads the current state (`systemctl --user is-active`), and if a
service is down it restarts it and **re-checks** whether it actually came up —
including reading each unit's `Restart=` policy to decide whether a restart is
even allowed/meaningful. Everything is dry-run-able with `-n/--dry-run`; real
runs log each action with an explicit `[fixed]` tag when a repair succeeded.

---

### 🧹 glassy-clean — interactive cleanup

**What it is.** The housekeeper: removes orphans, old caches, broken symlinks
and leftover build junk — always interactively, always asking first.

**How it works.** It walks through categories one by one:
1. **Orphaned packages** — `pacman -Qdtq` (dependencies no longer needed),
   shows the list, asks, then removes with `pacman -Rns`.
2. **Pacman package cache** — sizes `/var/cache/pacman/pkg` and prunes old
   versions via `paccache` (installs `pacman-contrib` if missing).
3. **Incomplete AUR builds** — leftover build directories.
4. **Broken symlinks** — dangling links across common locations.
5. **Unused Flatpak runtimes** — `flatpak` leftovers.

Each step measures what it frees, and nothing is deleted without a
per-category confirmation (`-y/--yes` for unattended runs).

---

### 💥 glassy-destroy — deep-purge stubborn packages

**What it is.** The sledgehammer for packages that refuse to uninstall
cleanly. Removes the package, its *unique* dependencies, configs, caches,
user data, autostart entries, desktop files and leftovers in common
locations.

**How it works.** Safety is the core design:
- A **hard denylist** of critical system packages that can never be destroyed
  (`--list-denied` shows it).
- **Default is a dry-run** — it shows exactly what *would* be removed.
- **Double confirmation** — you must type the package name to proceed.
- Before deletion it creates an **undo tarball** of the affected configs, so
  even a deep purge can be inspected afterwards.
- Package selection goes through **fzf**, so you pick from the real installed
  list instead of typo-ing a name.
- `--execute` is required to actually perform the removal.

---

### 📦 glassy-install — the smart universal installer

**What it is.** One command to install anything on GlassyOS — it figures out
*what* the argument is and picks the right method automatically.

**How it works.** The argument is dispatched by type:
- **Name (a package query)** → searches the official repos (`pacman -Ss`) and
  the AUR in parallel (AUR RPC v5 JSON search), shows the results side by side
  with repository + version + description, and installs the chosen one via
  pacman or the AUR helper.
- **git/HTTPS URL** → clones the repository and detects the build system: if
  it finds a `Makefile` it runs make; for Python projects it tries `pipx
  install .` (falling back to `pip install --user .`); otherwise it tells you
  to inspect and build manually.
- **Archive URL** (`.tar.gz`, `.tgz`, `.tar.xz`, `.tar.bz2`, `.zip`) →
  downloads and extracts into a temp folder for review, so you can move the
  binaries where they belong.
- **Anything else that looks like a binary URL** → downloads it straight to
  `~/.local/bin/<name>` and marks it executable.

---

### 🚀 glassy-boot-setup — GlassyOS boot animation (Plymouth)

**What it is.** Configures the boot experience: Plymouth + an image-based GOS
theme + silent boot.

**How it works.** The strategy is Plymouth with a custom GlassyOS theme and a
quiet, silent boot (no verbose kernel spam). As a bonus it **frees boot
space**: it removes `/boot/intel-ucode.img` because the microcode is already
embedded in the system's UKI via the microcode hook — a clean deduplication.
Subcommands: default (install), `--uninstall` (revert to previous boot
setup), `--status` (show current state) and `--trim`.

---

### ✨ glassy-splash — the animated GOS logo

**What it is.** The startup animation: a GlassyOS logo that draws itself at
Hyprland start (`exec-once = glassy-splash`).

**How it works.** A tiny GTK4 window on a dark background (`#05050e`) with pure
CSS animations — no video, no image assets. The individual GOS letters get a
`gos-letter` class and animate in (`visible`), pause, then animate out
(`gone`); a decorative line (`gos-line`) draws across the screen in sync. If
GTK4 isn't available it exits silently (`sys.exit(0)`) — the splash must never
break the session.

---

### 🖥️ glassy-flex — the tiled multi-TUI flex display

**What it is.** Opens a curated **multi-terminal dashboard** on its own
workspace: a 3×2 grid of live TUIs, each in its own terminal.

**How it works.** It lays out a fixed, hand-tuned arrangement (visible right in
the source as an ASCII blueprint):

```
┌──────────┬──────────┬──────────┐
│  lavat   │ fastfetch│ peaclock │
├──────────┼──────────┼──────────┤
│ pipes.sh │ unimatrix│ glassy-  │
│          ├──────────┤ lyrics   │
│          │   cava   │          │
└──────────┴──────────┴──────────┘
```

System stats (`fastfetch`), a clock (`peaclock`), audio visualizer (`cava`),
matrix rain (`unimatrix`), pipes (`pipes.sh`) — and `glassy-lyrics` right in
the middle of the action. `glassy-flex` opens the layout, `glassy-flex stop`
closes all flex windows again.

---

## GlassVibe apps (`apps/`)

**GlassVibe** — the closed media-app layer of GlassyOS. Each app lives in its
own folder with a complete `setup` installer (checks dependencies, installs to
`~/.local/bin`, creates config, deletes itself). Full in-depth documentation
lives in each app folder.

### 🎤 glassy-lyrics — synced lyrics as big terminal text

**What it is.** Karaoke-style lyrics in your terminal. Every word is rendered
through **Pillow into a bitmap** and drawn with terminal half-block characters
(`█ ▀ ▄`) — so ASCII, ß, Hangul, accents and emoji all look identical. Works
with **any MPRIS-capable player** (Spotify, VLC, mpv, Firefox/YouTube,
Chromecast players) via `playerctl` — if playerctl sees it, glassy-lyrics can
display its lyrics.

**How it works (short version).** It asks playerctl which track is playing,
fetches synced lyrics (LRCLIB, local `.lrc` files, and an optional
Musixmatch/NetEase fallback), then renders each line as huge terminal text by
rasterizing the font with Pillow and converting the bitmap to half-block
characters. When the source provides word-level timing, lyrics are
**highlighted word by word** in real time.

```bash
cd apps/glassy-lyrics
bash setup        # or: ./setup
```

→ Full technical documentation: [`apps/glassy-lyrics/README.md`](apps/glassy-lyrics/README.md)

### 🎵💡 glassy-light-sync — music-reactive RGB lighting

**What it is.** Syncs a Tuya Cloud RGB(W) strip to the music playing in your
room — driven by the **microphone**, not by player metadata, so it works with
any music source (speakers, headphones, everything).

**How it works (short version).** It records audio via `sounddevice`, splits
the spectrum with a **scipy FFT** into bass/mids/highs, and — engine **v13**
with **auto-calibration** — first measures your ambient noise floor for 2
seconds at startup, then only reacts to *relative* volume changes above that
threshold (cheap or auto-gained microphones stay reliable). Music triggers
color pulses pushed to the strip through the **Tuya Cloud API**; when the
music stops, the strip dims back to a quiet warm white.

```bash
cd apps/glassy-light-sync
bash setup        # or: ./setup
nano ~/.config/glassy-light-sync/tinytuya.json   # fill in Tuya credentials
glassy-light-sync --list-mics
glassy-light-sync --mic 2 --dry-run              # test without hardware
```

> 🔒 The real `tinytuya.json` is git-ignored — `tinytuya.example.json` is the
> public template. Never commit your Tuya keys.

→ Full technical documentation: [`apps/glassy-light-sync/README.md`](apps/glassy-light-sync/README.md)

---

## 🏗️ The GlassyOS ecosystem

| Project | Description |
|---|---|
| **glasstools** (this repo) | The closed system tools + GlassVibe apps of GlassyOS |
| gaur | GlassyOS AUR helper — paru-compatible, zero-dependency, with malware scan |

**GlassyOS releases next year.** This repository is where the distribution's
tooling is being built until then — closed, exclusive, and made for GlassyOS
only.

## 📄 License

[MIT](LICENSE) © GlassyOS

---

Built with 🧡 for GlassyOS.
