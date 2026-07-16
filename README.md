# waytime

Per-app and per-website screen time tracking for KDE Plasma (Wayland), with a CLI viewer.

## How it works

- `waytime-daemon` runs as a systemd user service. It injects a small KWin
  script (via the `org.kde.kwin.Scripting` D-Bus interface) that reports every
  window activation and title change back to the daemon. Every 5 seconds the
  daemon credits elapsed time to the focused app in a SQLite database at
  `~/.local/share/waytime/waytime.db`.
- For browser windows (Zen, Firefox, Chromium, etc.) the daemon also parses the
  site name out of the window title (e.g. "Video - YouTube — Zen Browser" →
  "YouTube") and keeps a per-website table. It's a heuristic, not a URL —
  pages whose titles omit the site name get odd labels.
- Tracking pauses while the screen is locked, and suspend gaps are not counted.
- If KWin restarts, the daemon re-injects the tracker script automatically.
- History is kept forever (the DB grows by a few KB per day).

Requires KDE Plasma 6 on Wayland — the daemon talks to KWin's scripting
interface, so other compositors (Hyprland, Sway, GNOME) would need their own
tracking backend.

## Install

From the AUR:

```
yay -S waytime        # or paru, or makepkg -si from a clone
systemctl --user enable --now waytime
```

This installs `waytime` (CLI) and `waytime-daemon` to `/usr/bin`, and the
user service to `/usr/lib/systemd/user/waytime.service`.

### Other distros (Kubuntu, Fedora KDE, openSUSE, …)

Any distro running KDE Plasma 6 on Wayland works. First make sure the two
Python D-Bus libraries are installed (they usually already are on KDE):

```
sudo apt install python3-dbus python3-gi     # Debian/Ubuntu/Kubuntu
sudo dnf install python3-dbus python3-gobject   # Fedora
sudo zypper install python3-dbus-python python3-gobject   # openSUSE
```

Then install to your user account (no root needed):

```
git clone https://github.com/ieatfood314/waytime.git
cd waytime
mkdir -p ~/.local/bin ~/.config/systemd/user
install -m755 waytime waytime-daemon ~/.local/bin/
sed "s|/usr/bin/waytime-daemon|$HOME/.local/bin/waytime-daemon|" waytime.service > ~/.config/systemd/user/waytime.service
systemctl --user daemon-reload
systemctl --user enable --now waytime
```

Make sure `~/.local/bin` is on your `PATH`. To uninstall:
`systemctl --user disable --now waytime`, then delete the three installed files.

## Usage

```
waytime                today's total and per-app breakdown
waytime yesterday      same, for yesterday
waytime week           per-day totals for the last 7 days
waytime history [N]    per-day totals for the last N days (default 14)
waytime apps [N]       per-app totals over the last N days (default 7)
waytime sites [N]      per-website totals over the last N days (default: today)
waytime 2026-07-01     breakdown for a specific date
```

## Service management

```
systemctl --user status waytime     # check the daemon
journalctl --user -u waytime -f     # follow its logs
systemctl --user restart waytime    # restart after an update
```

## Disclaimer

🤖 This project was vibe coded with Claude (Fable 5) — a human chose what to
build and verified it works; the AI wrote the code. Read it before you run it,
as you should with any stranger's daemon.
