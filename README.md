# glasstools 🛠️

**The closed system tooling of GlassyOS — thirteen hand-written system commands plus the GlassVibe media apps, built as one tightly integrated unit.**

glasstools is the toolbox that will ship with **GlassyOS**, a beautiful, safe,
low-maintenance Linux distribution built on top of Arch Linux. Everything here is
written from scratch — plain Python 3 and Bash, no frameworks, no bloat — and designed
around the GlassyOS architecture, its lifecycle and its philosophy. The source is
public to show the craft behind GlassyOS, but each tool is engineered for GlassyOS
first and only.

> 🔒 **Made exclusively for GlassyOS.** These are **not** generic public utilities.
> They are private systems of the distribution — an update stack, a rollback stack, a
> security layer, an onboarding tour, boot theming and the GlassVibe media apps —
> built as a coherent whole.

> 🗓️ **GlassyOS releases in 2027.** Until then, this repository is the living workshop
> of everything the distribution ships with. Every tool is developed here, every tool
> is exclusive to GlassyOS.

---

## Table of contents

- [Why glasstools?](#why-glasstools)
- [What's inside](#whats-inside)
- [Requirements](#requirements)
- [Installation](#installation)
- [Quick start](#quick-start)
- [The GlassyOS identity gate](#the-glassyos-identity-gate)
- [System tools (`bin/`)](#system-tools-bin)
  - [🔄 glassy-update](#-glassy-update--the-smart-updater-python-tui)
  - [⏪ glassy-downgrade](#-glassy-downgrade--roll-the-system-back-python-tui)
  - [👋 glassy-onboard](#-glassy-onboard--the-first-boot-welcome-tour-gtk4libadwaita)
  - [🛡️ glassy-checkup](#️-glassy-checkup--malware--integrity-scanner)
  - [🕵️ glassy-secure](#️-glassy-secure--real-time-download-scanner-daemon)
  - [🩺 glassy-doctor](#-glassy-doctor--system-health-check--auto-fix)
  - [🔧 glassy-fix](#-glassy-fix--automatic-self-repair)
  - [🧹 glassy-clean](#-glassy-clean--interactive-cleanup)
  - [💥 glassy-destroy](#-glassy-destroy--deep-purge-stubborn-packages)
  - [📦 glassy-install](#-glassy-install--the-smart-universal-installer)
  - [🚀 glassy-boot-setup](#-glassy-boot-setup--the-glassyos-boot-animation-plymouth)
  - [✨ glassy-splash](#️-glassy-splash--the-animated-gos-logo)
  - [🖥️ glassy-flex](#️-glassy-flex--the-tiled-multi-tui-flex-display)
- [GlassVibe apps (`apps/`)](#glassvibe-apps-apps)
  - [🎤 glassy-lyrics](#-glassy-lyrics--synced-lyrics-as-big-terminal-text)
  - [🎵💡 glassy-light-sync](#-glassy-light-sync--music-reactive-rgb-lighting)
- [How it works (architecture)](#how-it-works-architecture)
- [Configuration & file locations](#configuration--file-locations)
- [Examples & recipes](#examples--recipes)
- [Troubleshooting & gotchas](#troubleshooting--gotchas)
- [Security & privacy notes](#security--privacy-notes)
- [GlassyOS (2027) release note](#glassyos-2027-release-note)
- [Status & roadmap](#status--roadmap)
- [Credits & license](#credits--license)

---

## Why glasstools?

Mainstream distros hand you a pile of unrelated tools that were written by different
people, at different times, with different ideas about how a system should behave. The
result is a system that feels assembled rather than designed.

glasstools takes the opposite approach. Every command is part of **one ecosystem**:

- **One design language.** The same colour palette, the same `glassy-` prefix, the
  same `[toolname]` log prefixes, the same ✓ / ⚒ / ✗ status vocabulary across every
  tool. When you learn one, you can read the others.
- **One safety model.** Snapshots before anything destructive, dry-runs by default,
  hard denylists for critical packages, double confirmations, quarantine instead of
  deletion. The dangerous operations are the ones that ask twice.
- **One philosophy.** Few visible moving parts, no daemons you didn't ask for, no
  telemetry, no cloud. Everything runs locally, fast, and predictable.
- **Zero frameworks.** No Electron, no Qt, no TUI library. The updater draws itself
  with raw ANSI escape codes; the splash is pure CSS; the apps are single files.

The result is a system that behaves like a single, deliberate thing.

---

## What's inside

Two closed areas live in this repository:

| Area | Contents |
|---|---|
| [`bin/`](#system-tools-bin) | **13 system tools** — update, rollback, security, health, install, boot theming & more |
| [`apps/`](#glassvibe-apps-apps) | **GlassVibe** media apps — full user-facing tools with their own setup installer |

```
glasstools/
├── bin/                        # 13 system commands (plain Bash + Python 3)
│   ├── glassy-update           # smart updater TUI (+ hidden admin console)
│   ├── glassy-downgrade        # rollback TUI
│   ├── glassy-onboard          # first-boot GTK4/libadwaita welcome tour
│   ├── glassy-checkup          # malware & integrity scanner + quarantine
│   ├── glassy-secure           # real-time download scanner daemon
│   ├── glassy-doctor           # health check + auto-fix
│   ├── glassy-fix              # unattended self-repair
│   ├── glassy-clean            # interactive cleanup
│   ├── glassy-destroy          # deep-purge stubborn packages
│   ├── glassy-install          # universal installer
│   ├── glassy-boot-setup       # Plymouth boot animation
│   ├── glassy-splash           # animated GOS logo
│   └── glassy-flex             # tiled multi-TUI dashboard
└── apps/                       # GlassVibe media apps
    ├── glassy-lyrics/          # karaoke lyrics in the terminal
    └── glassy-light-sync/      # music-reactive RGB lighting
```

Each GlassVibe app lives in its own folder with a complete `setup` installer that
checks dependencies, installs to `~/.local/bin`, creates config and then deletes
itself.

---

## Requirements

The tools are written for **Arch Linux / GlassyOS** and assume a normal Arch user
session. There is **no build step** — everything is interpreted.

**Base:**

- **Bash 5+** and **Python 3.8+** (standard library only for the core tools).
- `pacman` (package management) and either a display session (Hyprland) for the GUI
  tools.

**Per-tool extras** (only pulled in when you use that tool):

| Tool | Extra dependencies |
|---|---|
| `glassy-update` | `reflector` (mirrors), `fzf` (specific package), `rate-mirrors` |
| `glassy-downgrade` | `fzf`, network access to `archive.archlinux.org` |
| `glassy-onboard` | GTK4 + libadwaita (PyGObject) |
| `glassy-checkup` | `clamav`, `rkhunter` (offered for auto-install) |
| `glassy-secure` | `inotify-tools`, `clamav`, `notify-send` |
| `glassy-doctor` / `glassy-fix` | nothing special (probes the running session) |
| `glassy-clean` | `pacman-contrib` (`paccache`), optionally `flatpak` |
| `glassy-destroy` | `fzf` |
| `glassy-install` | `fzf`, `jq`, `curl` |
| `glassy-boot-setup` | `plymouth`, `mkinitcpio`, an EFI/UKI setup |
| `glassy-splash` | GTK4 (silently skips if missing) |
| `glassy-flex` | `hyprctl` (Hyprland), `jq`, `kitty`, and the TUIs it launches |
| `glassy-lyrics` | `playerctl`, Pillow, a bold font |
| `glassy-light-sync` | `numpy`, `scipy`, `sounddevice`, `portaudio`, `tinytuya` |

The core identity gate expects a `glassyos-license` binary on `PATH` — see
[The GlassyOS identity gate](#the-glassyos-identity-gate).

---

## Installation

The system tools are single files. To make them available everywhere, copy them into
your user `PATH`:

```bash
git clone git@github.com:eliii-te/glasstools.git
cd glasstools

# install all 13 system tools
install -Dm755 bin/glassy-* ~/.local/bin/
```

Then verify one of them and optionally wire up the first-boot pieces:

```bash
glassy-update --version     # should print the tool version
glassy-doctor               # health check of your current session
glassy-onboard --force      # preview the welcome tour by hand
```

The **GlassVibe apps** ship with their own installer. Run it from inside the app
folder — it checks dependencies, installs to `~/.local/bin`, writes a default config
and then deletes itself:

```bash
cd apps/glassy-lyrics         && bash setup
cd ../glassy-light-sync       && bash setup
```

> **Note on the identity gate:** on a plain Arch machine (no `glassyos-license`), every
> tool exits immediately with code **77**. On real GlassyOS the licence is present and
> the gate passes silently. See below.

---

## Quick start

```bash
glassy-update                # open the update TUI (arrow keys, Enter to run)
glassy-update --full         # non-interactive: mirrors → repos → apps → glassyOS
glassy-downgrade             # open the rollback TUI (danger palette: red)
glassy-doctor                # check Bluetooth, Wi-Fi, audio, Waybar, Hyprland, …
glassy-doctor --no-fix       # diagnose only, touch nothing
glassy-fix --dry-run         # see what the auto-repair would do
glassy-clean                 # interactive cleanup (asks before every step)
glassy-install firefox       # search repos + AUR, pick, install
glassy-checkup --quick       # fast malware + heuristic scan
glassy-checkup --full        # ClamAV + rkhunter + package integrity
glassy-secure status         # is the real-time download scanner running?
glassy-flex                  # open the 3×2 multi-TUI dashboard
```

Typical first-run workflow for a maintainer on a fresh GlassyOS install:

```bash
glassy-onboard --first-run   # the welcome tour (also runs once at first login)
glassy-boot-setup            # install the Plymouth boot animation
glassy-update --full         # bring the whole system up to date
glassy-checkup --full        # baseline security scan
glassy-secure install        # turn on the always-on download scanner
```

---

## The GlassyOS identity gate

Every tool in this repository refuses to run without a valid GlassyOS licence. The very
first thing each script does is call the licenser's `gate` subcommand:

```bash
# Bash tools
if ! glassyos-license gate; then exit 77; fi

# Python tools
if subprocess.run(["glassyos-license", "gate"], stdout=DEVNULL).returncode != 0:
    sys.exit(77)
```

If the gate fails, the process exits with status **77** and does nothing else. This
keeps the closed tooling genuinely closed: the code is readable, but the behaviour
belongs to licensed GlassyOS machines.

- The **licenser itself** is the one exception — it must be able to run in order to
  validate everything else.
- On a real GlassyOS system the licence is present, so the gate passes silently and you
  never notice it.
- When hacking on the tools on a plain Arch box, either provide a valid
  `glassyos-license` on `PATH`, or work on the `.bak-gate` copies you find alongside the
  tools in this repo (the pre-gate versions kept for development).

---

## System tools (`bin/`)

The everyday system layer of GlassyOS — exclusive tooling that ships with the
distribution. All of them share the same design: coloured `[toolname]` log prefixes, a
`▸` step banner, and ✓ / ⚒ / ✗ status marks.

---

### 🔄 glassy-update — the smart updater (Python TUI)

**What it is.** The heart of the GlassyOS update experience. An interactive,
full-screen terminal UI with an animated splash, an arrow-key menu and real update
actions — every action wrapped in a **rolling snapshot backup**, so the system can
always be rewound afterwards. Version marker: `ALPHA:0.1`.

**How it works — the UI layer.** The tool talks to the terminal in raw mode
(`termios` + `tty`, via its own `RawMode` context manager) and reads arrow keys
directly instead of using a TUI framework. It renders an animated splash, a chromed
frame (`render_chrome`) with title/subtitle and a footer, and paints the menu from
scratch on every keypress. Everything degrades gracefully to plain output when no TTY
is available (piped output won't hang).

**How it works — snapshots.** Before *any* update, `make_snapshot()` creates a
timestamped folder (`YYYYMMDD-HHMMSS-<reason>`) under `/var/lib/glassyos/backups`, or
under `~/.local/share/glassy/backups` if the system path isn't writable (the tool
probes with a touch file and falls back automatically). A snapshot contains:

- `pkglist-explicit.txt` — `pacman -Qqe`, the packages you explicitly installed
- `pkglist-aur.txt` — `pacman -Qqm`, packages from the AUR
- `pkglist-all.txt` — `pacman -Qq`, every installed package
- `mirrorlist` — a copy of `/etc/pacman.d/mirrorlist`
- `glassyos-release` — the active GlassyOS version file

Only the **last 10 snapshots** are kept; older ones are pruned automatically.

**The menu.** Arrow keys move, Enter runs, Esc/`q` leaves:

| Item | What it does |
|---|---|
| **Update GlassyOS** | Pulls the next OS release: kernel, base packages, `glassy-*` scripts. |
| **Update mirrors** | Re-ranks and rewrites the pacman mirrorlist for the fastest sources. |
| **Update repos** | Refreshes the arch + blackarch package databases (`-Syy`). |
| **Update system applications** | Upgrades every installed package across all managers. |
| **Update everything** | Full sweep: mirrors → repos → applications → glassyOS. Slowest, safest. |
| **Update specific package** | Picks exactly one package via fuzzy search and updates it. |
| **Exit** | Leaves the updater. |

**The actions, in detail.**

- **Update GlassyOS** reads `release_manifest_url` + `channel` from the config
  (`/etc/glassyos/updater.conf` or `~/.config/glassy/updater.conf`), downloads the
  release **manifest** (JSON), compares the announced `version` against the installed
  one (`/etc/glassyos-release`), and — if newer — downloads `tarball_url`, **verifies
  its SHA-256** against the manifest's `sha256` (a mismatch aborts the update), then
  applies it with `tar --use-compress-program=unzstd -xf … -C /`, running
  `/opt/glassyos/install.sh` afterwards if present.
- **Update mirrors** runs `reflector --latest 20 --protocol https --sort rate --save
  /etc/pacman.d/mirrorlist` (falls back to `pacman -Syy` when reflector is missing).
- **Update repos** refreshes `archlinux-keyring`, then `blackarch-keyring` if
  blackarch is installed, then force-refreshes every database with `pacman -Syy`.
- **Update system applications** upgrades across **every** package manager it finds:
  `paru`/`yay` (or `pacman`) with `-Syu`, `flatpak update -y`, `pipx upgrade-all`,
  `rustup update`, `cargo install-update -a`, and `npm update -g` (only if the npm
  prefix is user-writable). Afterwards it lists any `.pacnew` files and tells you to
  review them with `pacdiff`.
- **Update specific package** lists `pacman -Qq` into **fzf** (rounded border, reverse
  layout) and re-installs the chosen package via `paru -S --needed` or
  `pacman -S --needed`.
- **Update everything** chains all of the above in one pass, isolating each step so one
  failure doesn't abort the rest.

**CLI flags.**

| Flag | Meaning |
|---|---|
| `--version` | Print the tool version and exit. |
| `--no-splash` | Skip the intro animation. |
| `--apps` | Run "update system applications" non-interactively. |
| `--full` | Run "update everything" non-interactively. |

**Admin/publish console.** Hidden behind a **secret trigger sequence** typed in the main
menu, plus the user's **login PIN** and an **admin password** — both stored as salted
SHA-256 hashes (`SALT = "glassyos-updater-v1"`). Three failed attempts abort. This
gates publish/release actions for the GlassyOS maintainer: a self-contained release
workflow living inside the updater. (The actual trigger string and hashes live in the
source; they are intentionally not repeated here.)

---

### ⏪ glassy-downgrade — roll the system back (Python TUI)

**What it is.** The power-user companion to `glassy-update`: an interactive TUI to
rewind parts of GlassyOS / Arch to an earlier state. It uses a red, "danger" colour
palette to underline that this is a power tool — rewinding a rolling-release system can
break package signatures, leave dangling dependencies and confuse pacman's database. It
warns loudly, but does not stop you. That's the point.

**The menu** offers two paths (plus Exit):

| Item | Description (from the tool itself) |
|---|---|
| **System downgrade** | "installs an older version of arch — WARNING: THIS IS ABSOLUTELY NOT RECOMMENDED" |
| **Specific downgrade** | "downgrades selected packets, execute with attention" |

**Path 1 — System downgrade.** A sub-menu lets you choose either:

1. **Arch — rewind to an archive snapshot.** The tool lists the last **24 months** of
   snapshots from `https://archive.archlinux.org/repos`, you pick a date, it writes a
   single-entry mirrorlist pointing at that archive snapshot and runs
   `sudo pacman -Syyuu --noconfirm` to roll the whole system back to that point in
   time. Afterwards it reminds you that you're now **pinned** to the archive and should
   run `glassy-update` later to return to current.
2. **GlassyOS — install an older release.** Reads available releases (local
   `manifest-*.json` files in `~/.local/share/glassy/releases`, or fetched from a
   `releases_catalog_url` in the config), lets you pick one, then re-applies it exactly
   like the updater: download the tarball, **verify the SHA-256**, extract to `/` and
   run `/opt/glassyos/install.sh`.

**Path 2 — Specific downgrade.** For a single package: it fuzzy-searches the installed
list with **fzf**, shows the current version, then searches the **local pacman cache**
(`/var/cache/pacman/pkg`) and queries the **Arch Linux archive**
(`archive.archlinux.org/packages/<first-letter>/<pkg>/`) for older versions. You pick a
version from the merged list (cache entries are local; archive entries are downloaded
first) and it installs with `sudo pacman -U --noconfirm`. If it succeeds it suggests
adding an `IgnorePkg = <pkg>` line so the version stays pinned.

**Safety.** Before anything destructive, both paths create the same snapshot backups as
`glassy-update`, so the pre-rollback state is always recoverable.

**CLI flags.**

| Flag | Meaning |
|---|---|
| `--version` | Print version. |
| `--no-splash` | Skip the intro animation. |
| `--system` | Run the system downgrade non-interactively. |
| `--specific` | Run the specific-package downgrade non-interactively. |

---

### 👋 glassy-onboard — the first-boot welcome tour (GTK4/libadwaita)

**What it is.** A fullscreen GUI welcome tour that runs automatically on the very first
GlassyOS session. It is wired in via Hyprland `exec-once` and gated by a completion
marker (`~/.config/glassy/onboard-done`), so it only appears once.

**How it works.** It's a GTK4 + libadwaita application with a smooth
`Adw.Carousel` of pages, carousel-indicator dots, and swipe/arrow navigation. A shared
`page_shell()` scaffold builds every page from three parts: a small *eyebrow* tag on
top, a big *title*, and a body block — all centred. Head/keyboard navigation moves the
carousel forward and backwards. On the last page, finishing writes the marker file.

**The pages.**

- **Welcome** — an animated intro with a Cairo-drawn triangle logo that pulses.
- **INTRODUCTION · "What is glassyOS"** — bullet rows explaining the philosophy, plus a
  separator and a closing statement.
- **THE TOOLBOX · "Ten commands. One system."** — a grid of cards, one per `glassy-*`
  command, each showing its glyph and name.
- **Per-tool pages** (`TOOL · n/N`) — one detailed card per tool with its glyph,
  name, tagline, description and example commands.
- **QUICKSHELL · "Mod + Ctrl summons widgets"** — key-chip rows describing the
  Quickshell popups.
- **WAYBAR · "Click the bar. It clicks back."** — a **mock status bar** drawn with CSS
  classes (fake workspace pills, title, tray items) with callout arrows pointing at the
  interactive parts.
- **Keyboard shortcut pages** — launch, window, workspace, screenshot and system
  shortcuts, each rendered as `key-chip` rows connected with little `+` separators.

Styling is inline CSS only — rounded cards, soft backgrounds, custom popup chrome, key
chips. No image assets; everything is drawn live.

**CLI flags.**

| Flag | Meaning |
|---|---|
| `--first-run` | Run only if onboarding has never been completed. |
| `--force` | Run even if completed; do not write the marker. |
| `--reset` | Delete the completion marker and exit. |

---

### 🛡️ glassy-checkup — malware & integrity scanner

**What it is.** The on-demand security scanner: ClamAV + rkhunter + its own heuristic
checks, with a built-in quarantine system. Runs a quick pass or a full system sweep.

**How it works.** It dispatches on a **mode** — `--quick`, `--full`, `--update`, or a
custom path argument — and each mode defines its own scan profile: which directories
ClamAV recurses over, where to hunt for suspicious SUID binaries, where to look for
executables in writable temp locations, whether to scan `$HOME` for hidden
executables, and whether rkhunter + a full `paccheck` package-integrity run are
included.

| Mode | Profile |
|---|---|
| `--quick` | Scans `~/Downloads`, `~/.local/bin` and `/tmp` with the heuristic hunts; skips ClamAV + rkhunter + paccheck. |
| `--full` | ClamAV system-wide **and** rkhunter **and** a package-integrity check, deepest heuristics. |
| `--update` | Refreshes virus signatures (`freshclam`) and rkhunter's database, then exits. |
| `<path>` | Custom: scans the given path only. |

**The heuristic checks** look for suspicious SUID binaries in user-writable or unusual
locations, executables in temp locations, hidden executables in `$HOME`, suspicious
cron jobs, unexpected user systemd services, and (in full mode) recently-modified
system binaries. Findings are recorded with a severity (`MEDIUM`, `WARNING`).

Missing tools (clamav, rkhunter) are offered for auto-install; the ClamAV signature
service is enabled and an initial `freshclam` run is triggered.

**Quarantine.** Threats are moved into a managed quarantine area
(`~/.local/share/glassy/quarantine`), stored **unreadable** (`chmod 000`) and logged to
`quarantine.log` (timestamp, reason, original path, ID).

| Command | Effect |
|---|---|
| `glassy-checkup --quarantine` | List quarantined files. |
| `glassy-checkup --restore <ID>` | Move a file back to its original path (asks first). |
| `glassy-checkup --purge-quarantine` | Permanently delete everything (type `purge` to confirm). |
| `glassy-checkup --test` | End-to-end self-test. |

`--test` drops the harmless, universally recognised **EICAR test string** into
`~/Downloads`, watches the scanner catch it and quarantine it — a full verification of
the pipeline.

---

### 🕵️ glassy-secure — real-time download scanner (daemon)

**What it is.** The always-on guardian: a small daemon that watches `~/Downloads` **in
real time** and scans every new file with ClamAV the moment it appears. Runs as a user
systemd service (`glassy-secure.service`).

**How it works.** It uses **inotify** (`inotifywait -m -e close_write -e moved_to`) to
watch the download folder. Each new file is scanned in the background:

- **ClamAV check** — `clamscan --no-summary --infected`. Detected threats are
  **auto-quarantined** (moved to `~/.local/share/glassy/quarantine`, `chmod 000`, logged)
  and the user gets a **critical desktop notification**.
- **Executable warning** — newly downloaded `.sh`, `.bin`, `.run`, `.AppImage`, `.deb`
  or `.rpm` files are flagged (a normal notification, *not* quarantined, since that's a
  legitimate case) with a hint to scan them via `glassy-checkup`.
- **MIME mismatch** — a file with a media/PDF/Office extension whose real MIME type is
  an executable or shared library is treated as a disguised executable and
  **quarantined** immediately.

Partially written files (`.part`, `.crdownload`, `.tmp`) are skipped so the scan only
happens once a download has finished.

**Subcommands.**

| Command | Effect |
|---|---|
| `glassy-secure install` | Writes and enables the user systemd unit (`Restart=on-failure`, 5 s). |
| `glassy-secure start` / `stop` | Start / stop the service. |
| `glassy-secure status` | Service status + the last 20 log lines. |
| `glassy-secure watch` | Run the watcher in the foreground (debug). |
| `glassy-secure uninstall` | Disable and remove the unit. |

The tool is one of the services that `glassy-doctor` and `glassy-fix` probe, and that
`glassy-fix` will restart if it dies.

---

### 🩺 glassy-doctor — system health check + auto-fix

**What it is.** The diagnostic: probes the critical subsystems of a GlassyOS session,
tries to repair what's broken, and prints a clear summary.

**How it works.** It checks each subsystem in turn and reports per check with **✓**
(ok), **⚒** (fixed) or **✗** (failed):

1. **Bluetooth** — is `bluetooth.service` active? rfkill-blocked?
2. **Wi-Fi** — is NetworkManager active? Wi-Fi radio on? rfkill-blocked?
3. **PipeWire / Audio** — are the PipeWire user units running?
4. **Waybar** — running (or startable)?
5. **Hyprland** — process running, `hyprctl` responding, and how many error entries
   are in the current Hyprland log.
6. **Autostart daemons** — probes for `awww`, `elephant`, `walker`, `dock`, `hypridle`
   and `cliphist`, and starts whatever is missing (if the binary exists).
7. **Polkit auth agent** — is an agent running? (needed for GUI sudo prompts).

When something is down it tries to start/repair it automatically — unless `--no-fix` is
given. At the end it prints a summary (total / ok / fixed / failed).

**Options.**

| Option | Meaning |
|---|---|
| `--no-fix` | Diagnose only; do not repair. |
| `-v`, `--verbose` | Show extra detail (e.g. last error log lines). |
| `-h`, `--help` | Usage. |

> Run it as your **normal user**, not root — user services need the user session. The
> tool refuses to run as root with a friendly warning.

---

### 🔧 glassy-fix — automatic self-repair

**What it is.** The automatic repair pass — the tool that actually *fixes* what
`glassy-doctor` found, aimed at unattended/scheduled use (e.g. a login hook or timer).
Every action logs an explicit `[fixed]` tag when a repair succeeded.

**The passes.**

1. **Stale pacman lock** — removes `/var/lib/pacman/db.lck` only if pacman isn't
   actually running (otherwise it leaves the valid lock alone).
2. **Failed user services** — `reset-failed` + `restart`, then re-checks whether the
   service actually came up.
3. **Enabled-but-not-running user services** — starts the ones that *should* be up.
4. **Failed system services** — resets and restarts them.
5. **Broken symlinks** — finds (`find -xtype l`) and removes dangling links in user
   dirs.
6. **Config validation** — checks syntax of user configs (e.g. Walker's TOML, JSON
   files) and reports problems.
7. **Package dependency integrity** — checks for broken dependencies.
8. **`.pacnew` files** — reports updated configs that still need merging.
9. **Glassy suite** — targeted checks for `elephant`, `walker` and `glassy-secure`,
   reading each unit's `Restart=` policy to decide whether a restart is meaningful.
10. **Quarantine notice** — informs you if items are pending in the security quarantine.

**Options.**

| Option | Meaning |
|---|---|
| `-n`, `--dry-run` | Print every action it *would* take, change nothing. |
| `-h`, `--help` | Usage. |

---

### 🧹 glassy-clean — interactive cleanup

**What it is.** The housekeeper: removes orphans, old caches, broken symlinks and
leftover build junk — always interactively, always asking first. Each step measures
what it frees and prints a final `df -h` summary.

**The categories** (one by one, each with its own confirmation):

1. **Orphaned packages** — lists `pacman -Qdtq` (dependencies no longer needed, up to
   20 shown) and removes them with `pacman -Rns`.
2. **Pacman package cache** — sizes `/var/cache/pacman/pkg`, dry-runs `paccache -dk3`
   (keeps the last 3 versions), then optionally runs `paccache -rk3` and
   `paccache -ruk0` (also purge uninstalled packages). Offers to install
   `pacman-contrib` if `paccache` is missing.
3. **AUR helper cache** — sizes `~/.cache/paru` or `~/.cache/yay` and optionally runs
   `paru -Sc` / `yay -Sc`.
4. **Incomplete AUR builds** — finds leftover `src/` and `pkg/` directories from failed
   `makepkg` runs and deletes them.
5. **Partial downloads** — finds `*.part` / `*.crdownload` files and offers to remove
   them.
6. **User cache** — sizes `~/.cache`, shows the 8 largest subdirectories, and can
   delete files not accessed in 30 days.
7. **Thumbnail cache** — clears `~/.cache/thumbnails`.
8. **Systemd journal** — shows `journalctl --disk-usage` and can vacuum logs to the
   last 2 weeks.
9. **Flatpak** — offers `flatpak uninstall --unused`.
10. **Broken symlinks** — scans `~/.local/bin`, `~/.local/share/applications` and
    `~/.config`.
11. **Trash** — offers to empty `~/.local/share/Trash`.

**Options.**

| Option | Meaning |
|---|---|
| `-y`, `--yes` | Auto-confirm every step (be careful). |
| `-h`, `--help` | Usage. |

---

### 💥 glassy-destroy — deep-purge stubborn packages

**What it is.** The sledgehammer for packages that refuse to uninstall cleanly. Removes
the package, its **unique** dependencies, configs, caches, user data, autostart entries,
desktop files and leftovers in common locations.

**Safety is the core design:**

- A **hard denylist** of critical system packages that can never be destroyed
  (`--list-denied` shows it, plus everything in the `base` group).
- **Default is a dry-run** — it shows exactly what *would* be removed.
- **Double confirmation** — you must type the package name to proceed.
- Before deletion it creates an **undo tarball** of the affected configs under
  `~/.local/share/glassy/destroy-undo`, so even a deep purge can be inspected
  afterwards.
- Package selection goes through **fzf**, so you pick from the real installed list
  instead of typo-ing a name.
- `--execute` is required to actually perform the removal.

**What the removal does**, once confirmed: analyses the cascade (`pacman -Rcs
--print-format '%n'`) so you see the unique dependencies that will go with it, stops the
package's systemd units, kills processes still using its files, runs
`sudo pacman -Rns --noconfirm` (falling back to `-Rdd` if the dependency check can't be
satisfied), deletes user-level leftovers, deletes system-level leftovers, refreshes the
desktop-file cache and finally verifies the package is gone.

**Usage.**

```bash
glassy-destroy                     # interactive picker (fzf)
glassy-destroy <package>           # show dry-run, ask for confirmation
glassy-destroy <name> --execute    # actually destroy (double-confirmation)
glassy-destroy --list-denied       # show protected system packages
```

If a protected system package is targeted, the tool refuses.

---

### 📦 glassy-install — the smart universal installer

**What it is.** One command to install anything on GlassyOS — it figures out *what* the
argument is and picks the right method automatically. Requires `fzf`, `jq`, `curl`,
`pacman`; AUR installs need `paru` or `yay`.

**The argument is dispatched by type:**

- **Name (a package query)** — searches the official repos (`pacman -Ss`) and the AUR
  **in parallel** (AUR RPC v5 JSON search, sorted by votes, top 40), shows the results
  side by side with source + repository + version + description, and installs the
  chosen one via `pacman -S --needed` or the AUR helper (`paru -S --needed`).
- **GitHub / git URL** — clones the repository (shallow) into `~/.local/src/<name>`
  (pulling if it already exists) and detects the build system: a `PKGBUILD` → `makepkg
  -si`; a `Makefile` → `make && sudo make install`; an `install.sh` → run it; a
  `Cargo.toml` → `cargo install --path .`; a `pyproject.toml` / `setup.py` → `pipx
  install .` (falling back to `pip install --user .`); otherwise it tells you to inspect
  and build manually.
- **Archive URL** (`.tar.gz`, `.tgz`, `.tar.xz`, `.tar.bz2`, `.zip`) — downloads and
  extracts into a temp folder for review, so you can move the binaries where they
  belong.
- **Anything else that looks like a binary URL** — downloads it straight to
  `~/.local/bin/<name>` and marks it executable.

**Examples.**

```bash
glassy-install AUR helper
glassy-install firefox
glassy-install https://github.com/user/tool
glassy-install https://example.com/tool.tar.gz
```

---

### 🚀 glassy-boot-setup — the GlassyOS boot animation (Plymouth)

**What it is.** Configures the boot experience: Plymouth + an image-based GOS theme +
silent boot.

**How it works.** The strategy is **Plymouth** with a custom GlassyOS theme and a quiet,
silent boot (no verbose kernel spam). It manages the Plymouth config
(`/etc/plymouth/plymouthd.conf`), the theme directory
(`/usr/share/plymouth/themes/glassyos`) and a set of silence kernel parameters
(`quiet`, `loglevel=3`, `splash`, `rd.systemd.show_status=false`,
`systemd.show_status=false`, `rd.udev.log_level=3`, `udev.log_level=3`,
`vt.global_cursor_default=0`).

As a bonus it **frees boot space**, a clean deduplication:

- removes `/boot/intel-ucode.img` (~15 MB) because the microcode is already embedded in
  the system's UKI via the `microcode` hook,
- removes stale EndeavourOS leftovers under `/boot/EFI/endeavouros`,
- **trims the kernel duplicate**: the kernel normally exists both in
  `/usr/lib/modules/$KVER/vmlinuz` (from the package) and in `/boot/vmlinuz-linux`
  (copied by an alpm hook). It rewrites `ALL_kver=` in
  `/etc/mkinitcpio.d/linux.preset` to resolve the package source dynamically, verifies
  the new resolution actually works, backs the old preset up and removes the copy.

**Subcommands.**

```bash
glassy-boot-setup             # install (Plymouth + GOS logo + silent boot)
glassy-boot-setup --trim      # only remove duplicates (intel-ucode + vmlinuz)
glassy-boot-setup --uninstall # revert to the previous boot setup
glassy-boot-setup --status    # show current state (themes, cmdline, EFI usage)
```

> Many of the log lines are in German (the maintainer's language) — the tool is
> internal to GlassyOS.

---

### ✨ glassy-splash — the animated GOS logo

**What it is.** The startup animation: a GlassyOS logo that draws itself at Hyprland
start. Add it to your startup config with:

```
exec-once = glassy-splash
```

**How it works.** A tiny GTK4 window on a dark background (`#05050e`) with **pure CSS
animations** — no video, no image assets. The individual **G**, **O** and **S** letters
carry a `gos-letter` class and fade in one after another (150 / 350 / 550 ms), a
decorative `gos-line` draws across in sync (750 ms), everything fades out at 1900 ms and
the window closes at 2700 ms. If GTK4 isn't available it exits silently
(`sys.exit(0)`) — the splash must never break the session.

---

### 🖥️ glassy-flex — the tiled multi-TUI flex display

**What it is.** Opens a curated **multi-terminal dashboard** on its own workspace
(workspace 8): a 3×2 grid of live TUIs, each in its own floating, semi-transparent
`kitty` window. The arrangement is fixed and hand-tuned (visible right in the source as
an ASCII blueprint):

```
┌──────────┬──────────┬──────────┐
│  lavat   │ fastfetch│ peaclock │
├──────────┼──────────┼──────────┤
│ pipes.sh │ unimatrix│  glassy- │
│          ├──────────┤  lyrics  │
│          │   cava   │          │
└──────────┴──────────┴──────────┘
```

- **lavat** — lava lamp animation
- **fastfetch** — system stats (refreshed every 30 s)
- **peaclock** — a big ASCII clock
- **pipes.sh** — the classic pipes screensaver
- **unimatrix** — Matrix rain (`-s 95 -f`)
- **cava** — the audio spectrum visualiser
- **glassy-lyrics** — the GlassVibe karaoke lyric display, right in the middle of the
  action

**How it works.** It reads the focused monitor and its reserved (bar) space from
`hyprctl monitors -j`, computes a 3-column / 2-row grid with gaps, then spawns each tile
with `hyprctl` window rules (float, size, move, noborder, noshadow, silent workspace)
and a `kitty` instance running the command. Each tile gets a `glassy-flex-<id>` window
title, which is how `stop` finds them again.

```bash
glassy-flex          # open the layout
glassy-flex stop     # close all flex windows again
```

Requires `hyprctl` (Hyprland), `jq` and `kitty`.

---

## GlassVibe apps (`apps/`)

**GlassVibe** — the closed media-app layer of GlassyOS. Each app lives in its own folder
with a complete `setup` installer (checks dependencies, installs to `~/.local/bin`,
creates config, deletes itself). Full in-depth documentation lives in each app folder.

### 🎤 glassy-lyrics — synced lyrics as big terminal text

**What it is.** Karaoke-style lyrics in your terminal. Every word is rendered through
**Pillow into a bitmap** and drawn with terminal half-block characters (`█ ▀ ▄`) — so
ASCII, ß, Hangul, accents and emoji all look identical. Works with **any MPRIS-capable
player** (Spotify, VLC, mpv, Firefox/YouTube, Chromecast players) via `playerctl`. If
playerctl sees it, glassy-lyrics can display its lyrics. Version: `1.7.0`.

**Features.**

- **Big text, rendered properly** — no figlet, no broken accents, no missing Hangul.
- **Live word-level karaoke** — from real word/syllable timing (Better Lyrics TTML /
  Spotify word-synced) instead of guessing; char-weighted interpolation only as a
  last-resort fallback on line-level LRCs.
- **Player-agnostic** — set `GLASSY_LYRICS_PLAYER` if your player isn't named `spotify`.
- **Five lyric sources race in parallel** (word-level wins over line-level):
  1. Local override LRC files (`~/.config/glassy-lyrics/lrc/`)
  2. Spotify's own color-lyrics API (needs your `sp_dc` cookie — optional)
  3. **Better Lyrics API** — TTML with word/syllable timing (the dataset Spicy Lyrics
     renders)
  4. LRCLIB (line-level)
  5. python-syncedlyrics (Musixmatch / NetEase) — optional fallback
- **Flicker-free** diff-aware rendering at 30 Hz; a background poller with
  monotonic-clock position extrapolation so the UI never blocks on playerctl.
- **Width-aware centering** with full CJK/wide-character support.
- Per-song offsets & language overrides via a JSON config.

**Usage.**

```bash
glassy-lyrics            # start with the default player (spotify)
glassy-lyrics --version  # or -V
glassy-lyrics --help     # or -h
# exit with Ctrl+C or Ctrl+\ — the screen is restored
```

**Environment variables.**

| Variable | Default | Description |
|---|---|---|
| `GLASSY_LYRICS_OFFSET` | `0.0` | Seconds added to playback position (positive = earlier). |
| `GLASSY_LYRICS_PLAYER` | `spotify` | playerctl player name. |
| `GLASSY_LYRICS_FALLBACK` | `1` | `0` disables the syncedlyrics fallback. |
| `GLASSY_LYRICS_HEIGHT` | `8` | Big-text height in terminal rows. |
| `GLASSY_LYRICS_DEBUG` | — | `1` keeps stderr visible (default: silenced). |
| `GLASSY_LYRICS_BOIDU_KEY` | — | Optional X-API-Key for the Better Lyrics API. |

**Configuration** lives in `~/.config/glassy-lyrics/`:

- `offsets.json` — per-song offsets, keys `"Artist|Title"` (case-insensitive). A bare
  number is an offset shorthand; an object can also set a lyric language.
  ```json
  {
    "Stray Kids|Chk Chk Boom": -0.4,
    "IU|Love wins all": { "offset": 0.2, "lang": "ko" }
  }
  ```
- `lrc/` — drop `<Artist> - <Title>.lrc` files here; they always win over remote
  sources.
- `sp_dc` — your Spotify browser cookie, if you want Spotify's own word-level lyrics.
  If it's rejected, the tool silently falls back to the other sources.

**How it works.** A background thread polls `playerctl` every 0.5 s and extrapolates
the position with a monotonic clock (the render loop never blocks). On track change, all
lyric sources are queried in parallel (threads + a 25 s race window); the first valid
hit wins and is cached. Each word is pre-rendered with Pillow to a bitmap, converted to
terminal half-block rows and cached (~5 ms/word). The renderer only redraws what
changed, writing ANSI cursor moves instead of clearing the screen → no flicker.

**Install.**

```bash
cd apps/glassy-lyrics
bash setup           # or: ./setup      (-y for non-interactive)
```

The installer detects your package manager (pacman / apt / dnf), installs the needed
system packages (python, Pillow, playerctl, a bold font), optionally installs the
`syncedlyrics` fallback via pip, copies the tool to `~/.local/bin/glassy-lyrics`, adds
`~/.local/bin` to `PATH` if needed, verifies the install and deletes itself.

→ Full technical documentation: [`apps/glassy-lyrics/README.md`](apps/glassy-lyrics/README.md)

---

### 🎵💡 glassy-light-sync — music-reactive RGB lighting

**What it is.** Syncs a Tuya Cloud RGB(W) strip to the music playing in your room —
driven by the **microphone**, not by player metadata, so it works with any music source
(speakers, headphones, everything). Engine **v13** with **auto-calibration**.

**How it works.** It records audio via `sounddevice`, splits the spectrum with a
**scipy FFT** into bass / mids / highs, and — with auto-calibration — first measures
your ambient noise floor for **2 seconds** at startup, then only reacts to *relative*
volume changes above that threshold (cheap or auto-gained microphones stay reliable).
Music triggers colour pulses pushed to the strip through the **Tuya Cloud API**; when
the music stops, the strip dims back to a quiet warm white.

> 💡 The Tuya Cloud API is a *cloud* round-trip (~300–500 ms), so this is a colour
> *mood* sync, not a sample-accurate beat light.

**Usage.**

```bash
glassy-light-sync --list-mics        # list input devices
glassy-light-sync --mic 2 --dry-run  # test without touching the hardware
glassy-light-sync --mic 2            # run (be quiet for 2 s calibration!)
glassy-light-sync --mic 2 -c /path/to/tinytuya.json
```

| Flag | Description |
|---|---|
| `-m`, `--mic INT` | Microphone index (see `--list-mics`). |
| `-c`, `--credentials PATH` | Path to `tinytuya.json`. |
| `-d`, `--dry-run` | Print colours instead of sending Tuya commands. |
| `-l`, `--list-mics` | List microphones and exit. |

Stop with `Ctrl+C` — the strip is restored to warm white on exit.

**Install & configure.**

```bash
cd apps/glassy-light-sync
bash setup           # or: ./setup      (-y for non-interactive)
nano ~/.config/glassy-light-sync/tinytuya.json   # fill in Tuya credentials
```

The installer installs the system packages (python, numpy, scipy, portaudio) and the
Python modules (`sounddevice`, `tinytuya`), copies the tool to
`~/.local/bin/glassy-light-sync` and creates the config from the example. The config
needs your Tuya IoT Platform credentials and device IDs:

```json
{
    "apiRegion": "eu",
    "apiKey": "YOUR_TUYA_API_KEY",
    "apiSecret": "YOUR_TUYA_API_SECRET",
    "gatewayDeviceId": "YOUR_GATEWAY_DEVICE_ID",
    "stripDeviceId": "YOUR_STRIP_DEVICE_ID"
}
```

> 🔒 The real `tinytuya.json` is **git-ignored** — `tinytuya.example.json` is the public
> template. Never commit your Tuya keys.

**Requirements.** A Tuya Cloud-enabled Zigbee gateway + RGB(W) strip (e.g. a Silvercrest
gateway + Paulmann MAXLEDS), Tuya IoT Platform credentials (iot.tuya.com, region `eu`).

→ Full technical documentation: [`apps/glassy-light-sync/README.md`](apps/glassy-light-sync/README.md)

---

## How it works (architecture)

glasstools is deliberately boring under the hood: small, focused programs that each do
one part of a system's lifecycle, sharing conventions instead of code.

| Layer | Implementation |
|---|---|
| **Language** | Plain **Bash 5** for the system glue, **Python 3 stdlib** for the TUIs and GTK apps. No third-party runtime for the core tools. |
| **UI — TUI** | Raw ANSI escape codes + `termios`/`tty` raw mode. The updater and downgrade tools render a chrome frame and paint the menu from scratch on each keypress. |
| **UI — GUI** | GTK4 + libadwaita (onboarding), Cairo (the animated logo), pure CSS (splash). |
| **Update model** | A JSON **manifest** (`version`, `tarball_url`, `sha256`) → SHA-256 verified tarball → `tar … -C /` + optional `/opt/glassyos/install.sh`. |
| **Rollback model** | **Snapshots** (package lists + mirrorlist + release file) before every action; plus full-system archive rewind and per-package cache/archive downgrade. |
| **Security model** | ClamAV + rkhunter + heuristics, a shared quarantine area, plus a real-time inotify daemon. |
| **Health model** | `glassy-doctor` (diagnose + fix) and `glassy-fix` (unattended repair) probing services, symlinks, configs and dependencies. |
| **Identity** | Every entry point calls `glassyos-license gate` and exits **77** on failure. |
| **Conventions** | `glassy-` binary prefix, `[toolname]` stderr log prefixes, ✓/⚒/✗ marks, a shared colour palette (sky/ice/hot/ok/warn/err). |

**Data flow of an OS update**, as a concrete example:

```
glassy-update  ──▶  make_snapshot()        →  /var/lib/glassyos/backups/…
      │
      ├─▶  load_config()                  →  release_manifest_url, channel
      ├─▶  GET manifest.json              →  { version, tarball_url, sha256 }
      ├─▶  compare vs /etc/glassyos-release
      ├─▶  download tarball               →  ~/.local/share/glassy/releases/
      ├─▶  sha256sum == manifest.sha256?  →  mismatch ⇒ reject
      └─▶  tar -C /  +  /opt/glassyos/install.sh
```

---

## Configuration & file locations

| Path | What lives there |
|---|---|
| `~/.config/glassy/updater.conf` | Updater config: `release_manifest_url`, `channel` (fallback: `/etc/glassyos/updater.conf`). |
| `/etc/glassyos-release` | Active GlassyOS version string (fallback: `~/.config/glassy/glassyos-release`). |
| `~/.local/share/glassy/backups/` | Snapshot folders (fallback when `/var/lib/glassyos/backups` isn't writable). |
| `~/.local/share/glassy/releases/` | Downloaded GlassyOS release tarballs + local `manifest-*.json`. |
| `~/.local/share/glassy/quarantine/` | Quarantined files (`chmod 000`) + `quarantine.log`. |
| `~/.local/share/glassy/logs/` | `secure.log` and other tool logs. |
| `~/.local/share/glassy/destroy-undo/` | Undo tarballs of configs removed by `glassy-destroy`. |
| `~/.config/glassy/onboard-done` | Marker: onboarding completed. |
| `~/.config/glassy-light-sync/tinytuya.json` | Tuya credentials for the light sync (git-ignored). |
| `~/.config/glassy-lyrics/` | `offsets.json`, `lrc/` overrides, `sp_dc` cookie. |
| `~/.config/systemd/user/glassy-secure.service` | The real-time scanner unit. |
| `~/.local/bin/` | Where all installed `glassy-*` commands live. |

**Ignored by git** (`.gitignore`): `__pycache__/`, `*.pyc`, and the real
`apps/glassy-light-sync/tinytuya.json` (credentials must never be committed).

---

## Examples & recipes

**Keep the system fresh in one shot.**

```bash
glassy-update --full          # mirrors → repos → apps → glassyOS, non-interactive
```

**Rewind a single misbehaving package.**

```bash
glassy-downgrade              # "Specific downgrade" → pick package → pick old version
# then, to pin it:
#   add  IgnorePkg = <package>  to /etc/pacman.conf
```

**Full-system rollback to a known-good date.**

```bash
glassy-downgrade              # "System downgrade" → "Arch" → pick an archive date
# later, return to current:
glassy-update --full
```

**Diagnose a broken session.**

```bash
glassy-doctor --no-fix -v     # diagnose everything, show detail, change nothing
glassy-fix --dry-run          # preview the automatic repairs
glassy-fix                    # run them for real
```

**Reclaim disk space safely.**

```bash
glassy-clean                  # step through every category, confirm each
glassy-clean --yes            # trust the defaults (careful)
```

**Hunt and quarantine malware.**

```bash
glassy-checkup --update       # refresh signatures
glassy-checkup --full         # deep scan
glassy-checkup --quarantine   # list what was caught
glassy-checkup --test         # verify the whole pipeline with EICAR
glassy-secure install         # keep watching ~/Downloads from now on
```

**Get rid of a package that won't die.**

```bash
glassy-destroy --list-denied     # confirm it isn't protected
glassy-destroy <package>         # see the dry-run
glassy-destroy <package> --execute
```

**Install literally anything.**

```bash
glassy-install firefox                          # repos + AUR, pick
glassy-install https://github.com/user/project  # clone + build
glassy-install https://example.com/tool.tar.gz  # download + extract
```

**Set up a fresh session's boot experience.**

```bash
glassy-boot-setup             # Plymouth theme + silent boot + free EFI space
glassy-boot-setup --status    # verify
```

**Movie night, hands-free.**

```bash
glassy-flex                   # launch the dashboard incl. big lyrics
glassy-light-sync --mic 2     # let the room react to the music
```

---

## Troubleshooting & gotchas

- **Everything exits with code 77.** That's the identity gate — there's no valid
  GlassyOS licence on `PATH`. Provide a working `glassyos-license`, or use the
  `.bak-gate` copies of the tools kept in this repo for development.
- **The updater TUI looks garbled.** It needs a real TTY. In a pipe it degrades to plain
  output; for scripting use `--apps` / `--full` instead of the interactive menu.
- **"no release manifest URL configured".** Set `release_manifest_url=` in
  `~/.config/glassy/updater.conf` (or `/etc/glassyos/updater.conf`) before using
  *Update GlassyOS*.
- **A release was rejected with a checksum mismatch.** Good — the tool refuses
  tampered downloads. Re-check the published `sha256` in your manifest.
- **`glassy-downgrade` needs `fzf`.** Without it, the picker modes print an install
  hint and bail. Same for `glassy-destroy` / `glassy-install`.
- **After a system downgrade you stay pinned to the archive.** That's intentional — run
  `glassy-update --full` to return to current packages.
- **`glassy-clean` asks about big things.** `paccache -ruk0` and journal vacuuming can
  remove a lot; read each prompt. Emptying the trash is permanent.
- **`glassy-destroy` refuses a package.** It's in the hard denylist or the `base` group —
  by design. Use `pacman -Rns` for normal removals; this tool is the last resort.
- **`glassy-secure` isn't scanning.** Is `inotify-tools` installed? Check
  `glassy-secure status` and `~/.local/share/glassy/logs/secure.log`. It only watches
  `~/Downloads` by default.
- **ClamAV signatures are stale.** Run `glassy-checkup --update`, or enable the
  `clamav-freshclam` service (the scanner offers to do this for you).
- **`glassy-doctor` complains it's running as root.** Run it as your normal user — user
  services live in your session, not root's.
- **The splash doesn't show.** It silently exits when GTK4 isn't available, and it's pure
  cosmetic — it must never break the session. Verify it's wired as
  `exec-once = glassy-splash`.
- **`glassy-boot-setup --trim` changed my kernel preset.** It backs the original up to
  `~/.local/share/glassy/boot-backup/linux.preset.pretrim.bak` and verifies the new
  resolution before removing `/boot/vmlinuz-linux`. Revert with `--uninstall` if needed.
- **`glassy-flex` opens nothing.** It needs `hyprctl` (Hyprland), `jq` and `kitty`, plus
  whichever of the TUIs you actually have installed. It also reserves space for Waybar
  automatically.
- **glassy-lyrics shows plain text instead of big block letters.** No compatible bold
  font was found — install a bold font (Noto CJK / DejaVu); the search list is at the
  top of the script.
- **glassy-lyrics says "No track playing".** Check `playerctl status`; if you use a
  non-Spotify player, run `GLASSY_LYRICS_PLAYER=vlc glassy-lyrics` (or `mpv`, `firefox`,
  …).
- **glassy-light-sync: "Baseline is very high".** Your mic is overdriven — lower the
  gain (`alsamixer` / `pavucontrol`) or choose another device with `--mic`.
- **glassy-light-sync does nothing, but `--dry-run` prints colours.** The device IDs in
  `tinytuya.json` are wrong, or the strip isn't paired to the gateway.
- **Never commit `tinytuya.json`.** It's git-ignored for a reason. Use
  `tinytuya.example.json` as the template.

---

## Security & privacy notes

- **Local-first, by design.** The core tools do no telemetry, no analytics and no
  phoning home. Network access is limited to what a tool *needs*: the release manifest /
  tarball, the Arch mirror list and archive, the AUR RPC, and (for light-sync) the Tuya
  Cloud API you configure yourself.
- **Verified updates.** `glassy-update` and `glassy-downgrade` verify the **SHA-256** of
  every downloaded release before applying it, and reject anything that doesn't match.
- **Destructive actions are guarded.** Snapshots before changes, dry-runs by default,
  hard denylists, double confirmations and undo tarballs are the standard here — the
  dangerous commands are the ones that ask twice.
- **Quarantine, not deletion.** The security tools move threats to a `chmod 000`
  quarantine and keep a log, so nothing is silently destroyed and everything is
  auditable.
- **The identity gate is a licensing boundary, not a security boundary.** It is enforced
  by a separate `glassyos-license` binary; the tool sources themselves are public.
- **The admin console is gated**, not just tucked away — a secret sequence, a login PIN
  and an admin password (both salted SHA-256), with a three-strike lockout.
- **Credentials stay out of git.** The only secret a tool needs from you (Tuya keys) is
  git-ignored, and a public example is provided.
- **The network tools scan local data only** unless you point them at a custom path.
- **Run the health tools as your user**, not root — and never `pkill -f "<pattern>"`
  where the pattern appears in your own command line (it will kill your own shell).

---

## GlassyOS (2027) release note

> 🗓️ **GlassyOS is scheduled for release in 2027.**

This repository is where the distribution's entire tooling is being built until then —
update stack, rollback stack, security layer, onboarding, boot theming and the
GlassVibe media apps. It is **closed, exclusive and made for GlassyOS only**:

- These are not generic utilities you drop into another distro; they are wired into
  GlassyOS's lifecycle, its services and its identity gate.
- The source is public so you can see **how** GlassyOS is built — the craft, the
  conventions, the philosophy — not to be repackaged elsewhere.
- When GlassyOS 2027 arrives, this repository is what its users will actually run.

If you're curious about the project, follow along — but treat everything here as a
work-in-progress workshop rather than a general-purpose toolkit.

---

## Status & roadmap

**Status: pre-release / active development (the tools self-describe as `ALPHA:0.1`).**

Works and is exercised today:

- the full updater menu with snapshot backups and checksum-verified releases,
- system + specific + GlassyOS downgrade paths,
- the security stack (checkup, real-time secure daemon, shared quarantine),
- health checks and automated repair,
- cleanup, deep-purge and universal install,
- onboarding tour, boot theming and the animated splash,
- both GlassVibe apps (lyrics 1.7.0, light-sync v13).

Ideas / not yet done:

- a user-facing GUI for the updater and downgrade flows,
- stronger release signing (beyond SHA-256 checksums),
- a unified settings app for all `glassy-*` configuration,
- more GlassVibe apps and tighter Quickshell/Waybar integration,
- packaging the whole toolbox as a set of Arch packages.

Contributions and ideas are welcome — open an issue.

---

## Credits & license

glasstools is part of the **GlassyOS** project.

Everything in this repository is **hand-written**: plain Bash and Python 3, plus GTK4 /
libadwaita / Cairo for the GUI pieces. There are no third-party application
dependencies to bundle — external tools (`pacman`, `reflector`, `fzf`, `clamav`,
`rkhunter`, `plymouth`, `playerctl`, `kitty`, the TUI toys used by `glassy-flex`, and
so on) are used as **external programs**, not vendored code.

A few components are used at runtime when present:

- **ClamAV** — https://www.clamav.net/ (GPL-2.0) — virus scanning.
- **rkhunter** — https://rkhunter.sourceforge.net/ (GPL-2.0) — rootkit hunting.
- **Pillow** — https://python-pillow.org/ (MIT-CMU) — bitmap rendering in glassy-lyrics.
- **Tuya / tinytuya** — https://github.com/jasonacox/tinytuya (MIT) — light control.
- **LRCLIB / Better Lyrics / Musixmatch / NetEase** — lyric data providers.

The tools and apps themselves are written from scratch and carry the project's own
license.

**[MIT](LICENSE)** © GlassyOS

---

Built with 🧡 for GlassyOS.
