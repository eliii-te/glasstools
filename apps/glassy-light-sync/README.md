# glassy-light-sync 🎵💡

Music-reactive RGB lighting — syncs a Tuya Cloud RGB(W) strip to whatever is
playing, using the **microphone** and FFT analysis.

Part of the **GlassVibe** apps in the GlassyOS tool suite. Engine v13 with
**auto-calibration**: at startup it measures your ambient noise floor for 2 s,
then only reacts to *relative* volume changes — so cheap or auto-gained
microphones work reliably.

## How it works

1. Record audio from the microphone (sounddevice + scipy FFT)
2. Split the spectrum into bass / mids / highs
3. On music (volume above the calibrated threshold): pick a color from a
   palette and push it to the strip via the Tuya Cloud API
4. When the music stops: dim the strip back to a quiet warm white

> 💡 The Tuya Cloud API is a *cloud* round-trip (~300–500 ms), so this is a
> color *mood* sync, not a sample-accurate beat light.

## Requirements

- Linux with a working input device (ALSA/PulseAudio)
- Python 3 + `numpy`, `scipy`, `sounddevice`, `tinytuya`
- A Tuya Cloud-enabled Zigbee gateway + RGB(W) strip (e.g. Silvercrest
  gateway + Paulmann MAXLEDS)
- Tuya IoT Platform credentials (iot.tuya.com, region `eu`)

## Install

```bash
bash setup          # or: ./setup  (-y for non-interactive)
```

The installer:
1. installs system packages if missing (python, numpy, scipy, portaudio)
2. installs `sounddevice` + `tinytuya` via pip if missing
3. copies the tool to `~/.local/bin/glassy-light-sync`
4. creates `~/.config/glassy-light-sync/tinytuya.json` from the example
5. deletes itself

Then edit the config:

```bash
nano ~/.config/glassy-light-sync/tinytuya.json
```

```json
{
    "apiRegion": "eu",
    "apiKey": "YOUR_TUYA_API_KEY",
    "apiSecret": "YOUR_TUYA_API_SECRET",
    "gatewayDeviceId": "YOUR_GATEWAY_DEVICE_ID",
    "stripDeviceId": "YOUR_STRIP_DEVICE_ID"
}
```

> 🔒 The real `tinytuya.json` is git-ignored — never commit your keys.
> Use `tinytuya.example.json` as the public template.

## Usage

```bash
glassy-light-sync --list-mics        # list input devices
glassy-light-sync --mic 2 --dry-run  # test without touching the hardware
glassy-light-sync --mic 2            # run (be quiet for 2 s calibration!)
glassy-light-sync --mic 2 -c /path/to/tinytuya.json
```

Options:

| Flag | Description |
|---|---|
| `-m, --mic INT` | microphone index (see `--list-mics`) |
| `-c, --credentials PATH` | path to `tinytuya.json` |
| `-d, --dry-run` | print colors instead of sending Tuya commands |
| `-l, --list-mics` | list microphones and exit |

Stop with `Ctrl+C` — the strip is restored to warm white on exit.

## Troubleshooting

- **"Baseline is very high"** → mic overdriven, lower the gain
  (`alsamixer` / `pavucontrol`) or pick another device with `--mic`
- **Tuya connection fails** → gateway online? keys correct? region `eu`?
  API key expired? Refresh it in the Tuya IoT Platform
- **Nothing happens, but dry-run prints colors** → device IDs in the config
  are wrong, or the strip is not paired to the gateway

## License

MIT © GlassyOS — part of the [glassy-tools](../) suite.
