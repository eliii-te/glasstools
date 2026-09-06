# glassy-tools 🛠️

The system tooling suite of **GlassyOS** — a Linux distribution built on top of
Arch Linux, designed to be beautiful, safe and low-maintenance.

Every tool in this repository is **self-contained, hand-written and
home-grown**. No frameworks, no bloat — plain Python 3 and Bash, the way
system tools should be.

Part of the **GlassyOS** ecosystem — the tools are built for GlassyOS first,
but every script is standalone and runs on any Arch-based system.

## 📦 The tools

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

## 🚀 Install

```bash
# copy the tools into your PATH
install -m755 bin/glassy-* ~/.local/bin/
```

Requirements depend on the tool — Python 3 for the Python tools, GTK4 +
libadwaita for `glassy-onboard`, ClamAV for `glassy-checkup`/`glassy-secure`,
Plymouth for `glassy-boot-setup`.

## 🏗️ Part of the GlassyOS ecosystem

| Project | Description |
|---|---|
| **glassy-tools** (this repo) | System tooling suite |
| glassy-lyrics | Synced lyrics as big terminal text (karaoke style) |
| gaur | GlassyOS AUR helper — paru-compatible, zero-dependency, with malware scan |

## 📄 License

[MIT](LICENSE) © GlassyOS

---

Built with 🧡 for GlassyOS.
