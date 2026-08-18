# DoSquats for Linux

> A Fedora/GNOME spinoff of **[joeljani/DoSquats](https://github.com/joeljani/DoSquats)**
> by Joel Jani — all credit for the idea goes there. Go star the original.

Clock-aligned reminders to get up and move.

The original is a macOS menu bar app written in Swift/AppKit, which doesn't port —
so this rebuilds the same behavior on native Linux pieces: a **systemd user timer**
for scheduling and a **GTK4** fullscreen overlay (or a plain notification) for the
reminder itself. It's an independent reimplementation, not a fork; no original
source is reused.

## Install

```
./squats install
```

Writes `~/.config/systemd/user/dosquats.{service,timer}`, enables the timer, and
starts it. User timers start at login, so that also covers launch-at-login.

No dependencies beyond a stock Fedora GNOME install: `python3-gobject`, `gtk4`,
`notify-send` (libnotify), `paplay` (pipewire-utils).

## Use

```
squats status             current settings and next fire time
squats test               show a reminder right now
squats on | off           enable / disable reminders
squats skip               skip the next reminder only

squats every <minutes>    5–60, in steps of 5
squats reps <count>       how many reps to ask for
squats exercise <name>    e.g. pushups, stretches
squats style overlay|banner
squats sound on|off
```

Config lives in `~/.config/dosquats/config.json`; every key is editable there too,
including `overlay_timeout`, `sound_file`, and `skip_when_locked`.

## How it maps to the original

| DoSquats (macOS) | Here |
|---|---|
| Clock-aligned interval | `OnCalendar=*:0/20` — fires at :00, :20, :40 |
| Missed reminders after sleep | `Persistent=true` — fires on resume |
| Full-screen blocking overlay | GTK4 fullscreen window, dismiss with Esc/Space/click |
| Banner style | `notify-send` |
| Sound | `paplay` on a freedesktop sound |
| Menu bar icon + toggles | **Not ported** — GNOME has no native tray. Use the CLI. |
| Launch at login | `systemctl --user enable` |

### Interval alignment

Intervals that divide 60 (5, 10, 15, 20, 30, 60) align perfectly to the clock.
Others (25, 40, 50…) restart their cycle each hour — `squats every` warns when
you pick one.

### Screen lock

If the screen is locked, the reminder is skipped rather than queued, so overlays
don't stack up while you're away. Set `"skip_when_locked": false` to disable.

## Uninstall

```
./squats uninstall
```

Removes the units; leaves `~/.config/dosquats/config.json` in place.

## License

MIT, same as the original.
