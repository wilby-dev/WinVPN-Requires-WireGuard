# Simple VPN

A tiny Windows app that gives the official [WireGuard](https://www.wireguard.com/) client a one-click **Connect / Disconnect** interface.

> **What this is (and isn't):** Simple VPN contains no VPN server and no encryption of its own. It controls the official WireGuard client, and you supply the WireGuard config from a VPN provider or your own server.

## Features

- Import a WireGuard `.conf` file with a file picker
- One-click Connect and Disconnect
- Live status indicator (green when connected), refreshed every second, so it stays accurate if you change the tunnel from WireGuard itself
- Checks that WireGuard is installed and links to the download if it isn't
- Single small `.exe` (~50 KB), no installer, no extra dependencies

## Requirements

- Windows 10 or 11 (64-bit)
- [WireGuard for Windows](https://www.wireguard.com/install/) installed in the default location (`C:\Program Files\WireGuard`)
- A WireGuard `.conf` file, from:
  - a provider that offers WireGuard configs (for example Mullvad, Proton VPN or IVPN), or
  - your own WireGuard server

## Usage

1. Install WireGuard for Windows.
2. Run `SimpleVPN.exe` and accept the Administrator prompt. Windows requires admin rights to create a network tunnel.
3. Click **Import config...** and choose your `.conf` file.
4. Click **Connect**. The status turns green when the tunnel is up.
5. Click **Disconnect** when you're done.

The app remembers your last imported config, so next time you only need to click Connect.

## How it works

Simple VPN is a thin wrapper around WireGuard's command line:

| Action     | Command run                                      |
|------------|--------------------------------------------------|
| Connect    | `wireguard.exe /installtunnelservice <config>`   |
| Disconnect | `wireguard.exe /uninstalltunnelservice <name>`   |
| Status     | Queries the `WireGuardTunnel$<name>` Windows service |

The tunnel name comes from your config's filename (letters, digits and `_ = + . -`, up to 15 characters).

## Security notes

- A WireGuard config contains a **private key**. Simple VPN copies the config you import to `%APPDATA%\SimpleVPN\`. Don't share that folder.
- The executable is **unsigned**. Windows SmartScreen may warn about an unknown publisher: click **More info**, then **Run anyway**.
- A VPN only protects traffic as far as the VPN server. Its privacy is only as good as the provider or server you connect to.

## Troubleshooting

| Problem | Fix |
|---|---|
| "WireGuard not installed" | Install it from wireguard.com/install, then reopen the app |
| "Could not start the tunnel" | Make sure the app is running as Administrator and the config is valid |
| "Doesn't look like a WireGuard config" | The file needs `[Interface]` (with `PrivateKey`) and `[Peer]` sections |
| Import button is greyed out | Disconnect first; configs can only be changed while disconnected |
| Tunnel name rejected | Rename the `.conf` to use only letters, digits and `_ = + . -` |

## Building from source

Cross-compile from Linux with MinGW-w64 (the project is `simplevpn.c`, `app.rc` and `app.manifest`):

```bash
x86_64-w64-mingw32-windres app.rc -O coff -o app.res.o
x86_64-w64-mingw32-gcc -O2 -Wall -mwindows -static -s simplevpn.c app.res.o \
  -o SimpleVPN.exe -lgdi32 -luser32 -lshell32 -lcomdlg32 -ladvapi32
```

`app.manifest` requests Administrator rights and enables modern Windows visual styles.

## Project layout

```
SimpleVPN/
├── SimpleVPN.exe   # the app
└── README.txt      # quick-start
```
