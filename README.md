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
squats snooze [minutes]   quiet now, remind again later
squats snooze cancel      drop a pending snooze

squats every <minutes>    5–60, in steps of 5
squats offset <minutes>   shift off the clock, e.g. -5 or +5
squats reps <count>       how many reps to ask for
squats exercise <name>    e.g. pushups, stretches
squats style overlay|banner
squats sound on|off
```

Config lives in `~/.config/dosquats/config.json`; every key is editable there too,
including `overlay_timeout`, `sound_file`, `skip_when_locked`, and `tray_icon`.

## Offset

Reminders land on the clock, which is exactly when meetings start. An offset
shifts every fire time, so `squats offset -5` on an hourly interval moves the
reminder to `:55` — before the call, not on top of it — and `+5` moves it to
`:05`. It applies to any interval: at 20 minutes, `-5` fires at `:15, :35, :55`.

The offset rewrites the timer's `OnCalendar=` as an explicit minute list, so a
negative offset wraps into the previous hour instead of drifting.

## Snooze

Reminders carry snooze buttons — on the overlay, and on a notification banner as
actions. Snoozing takes a deliberate mouse click: no key triggers it, and every
dismiss key (`Esc`, `Space`, `Return`, `q`) closes the reminder as it always did.
Snoozing does two things: it silences the
regular reminders that fall inside the window, and it schedules one reminder for
the end of it via a transient systemd timer. Defaults are 10 and 35 minutes —
long enough to see out the rest of a meeting — and they're the `snooze_options`
list in the config.

`squats status` shows a pending snooze, and `squats snooze cancel` drops it.

## Panel icon (the menu bar equivalent)

GNOME removed the system tray in 3.26 and never replaced it, so the icon needs
the AppIndicator extension:

```
sudo dnf install gnome-shell-extension-appindicator
./squats tray-install
```

Log out and back in once if no icon appears — that's the extension activating,
not the app failing.

You get a barbell panel icon whose menu mirrors the macOS one: next fire time,
enable/disable, skip next, snooze, interval, offset, reps, exercise,
overlay/banner, sound, and "Remind me now".

Reps and exercise offer presets plus a **Custom…** entry for anything else. The
submenu headers show the current value (`Reps (20)`), and a value set from the
CLI that isn't a preset shows up on the Custom… entry rather than being lost.

`tray-install` also copies `icons/dosquats-symbolic.svg` into your user icon
theme, which is what lets GNOME recolor it for light and dark panels. Set
`tray_icon` in the config to any other themed icon name if you'd rather — e.g.
`emoji-activities-symbolic` (a ball) or `alarm-symbolic`.

The tray is a **control panel, not a scheduler** — the systemd timer still fires
the reminders. Both read and write the same config file and the tray watches it
for changes, so the CLI and the menu stay in sync in both directions.

`./squats tray-uninstall` removes the icon; reminders keep running.

## How it maps to the original

| DoSquats (macOS) | Here |
|---|---|
| Clock-aligned interval | `OnCalendar=*:0/20` — fires at :00, :20, :40 |
| Missed reminders after sleep | `Persistent=true` — fires on resume |
| Full-screen blocking overlay | GTK4 fullscreen window, dismiss with Esc/Space/click |
| Banner style | `notify-send` |
| Sound | `paplay` on a freedesktop sound |
| Menu bar icon + toggles | Panel icon via AppIndicator — see below |
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
