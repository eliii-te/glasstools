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

Everything here is **self-contained, hand-written and home-grown**. No
frameworks, no bloat — plain Python 3 and Bash, the way system tools should
be. Two closed areas live in this repository:

| Area | Contents |
|---|---|
| [`bin/`](#system-tools) | System tools — update, health, security, installer & more |
| [`apps/`](#glassvibe-apps) | **GlassVibe** apps — full user-facing tools with their own setup installer |

---

## System tools (`bin/`)

The everyday system layer of GlassyOS — exclusive GlassyOS tooling that
ships with the distribution. Install by copying `bin/glassy-*` into your
PATH (`~/.local/bin`) or run them directly.

| Tool | Language | What it does |
|---|---|---|
| `glassy-update` | Python | Interactive TUI smart updater — animated splash, arrow-key menu, rolling update actions, hidden admin/publish console |
| `glassy-downgrade` | Python | Interactive TUI for rolling parts of GlassyOS / Arch back to an earlier state (companion to `glassy-update`) |
| `glassy-onboard` | Python | Fullscreen GTK4 + libadwaita welcome tour with Cairo-animated graphics — first-boot onboarding |
| `glassy-checkup` | Bash | Malware & integrity scanner — ClamAV + rkhunter + heuristic checks, supports quarantine |
| `glassy-secure` | Bash | Real-time download folder scanner — watches `~/Downloads` via inotify, ClamAV-scans new files, auto-quarantines |
| `glassy-doctor` | Bash | System health check + auto-fix — probes Bluetooth, Wi-Fi, PipeWire, Waybar, Hyprland, autostarts, polkit |
| `glassy-fix` | Bash | Automatic system health check and self-repair |
| `glassy-clean` | Bash | Interactive cleanup — orphans, old caches, broken symlinks, incomplete builds |
| `glassy-destroy` | Bash | Deep-purge a package that won't uninstall cleanly — package, unique deps, configs, caches, user data |
| `glassy-install` | Bash | Smart universal installer — searches pacman + AUR, handles git/curl URLs, picks the right method |
| `glassy-boot-setup` | Bash | Plymouth boot animation setup — image-based GOS theme + silent boot |
| `glassy-splash` | Python | Animated GOS logo, shown at Hyprland startup |
| `glassy-flex` | Bash | Tiled multi-TUI flex display on a dedicated workspace (3×2 grid) |

Install:

```bash
install -m755 bin/glassy-* ~/.local/bin/
```

---

## GlassVibe apps (`apps/`)

**GlassVibe** — the closed app layer of GlassyOS. Each app lives in its own
folder with a complete `setup` installer (checks dependencies, installs to
`~/.local/bin`, creates config, deletes itself).

### 🎤 glassy-lyrics — synced lyrics as big terminal text

Karaoke-style lyrics in your terminal: every word is rendered through Pillow
into a bitmap and drawn with half-block characters (`█ ▀ ▄`) — ASCII, ß,
Hangul, accents and emoji all look the same. Works with **any** MPRIS-capable
player (Spotify, VLC, mpv, …) via playerctl.

```bash
cd apps/glassy-lyrics
bash setup        # or: ./setup
```

### 🎵💡 glassy-light-sync — music-reactive RGB lighting

Syncs a Tuya Cloud RGB(W) strip to music picked up by your microphone (FFT
analysis, v13 auto-calibration engine). Reacts to relative volume changes
above your measured noise floor; dims the strip when music stops.

```bash
cd apps/glassy-light-sync
bash setup        # or: ./setup
# then fill in your Tuya credentials:
nano ~/.config/glassy-light-sync/tinytuya.json
glassy-light-sync --list-mics
glassy-light-sync --mic 2 --dry-run   # test without hardware
```

> 🔒 The real `tinytuya.json` is git-ignored — `tinytuya.example.json` is the
> public template. Never commit your Tuya keys.

---

## 🏗️ GlassyOS ecosystem

| Project | Description |
|---|---|
| **glasstools** (this repo) | The closed system tools + GlassVibe apps of GlassyOS |
| gaur | GlassyOS AUR helper — paru-compatible, zero-dependency, with malware scan |

## 📄 License

[MIT](LICENSE) © GlassyOS

---

Built with 🧡 for GlassyOS.
