---
name: local-wsl-env
description: >-
  This WSL machine's host-specific execution environment; all commands and
  paths exist only inside WSL. Documents passwordless sudo for a fixed
  whitelist of maintenance commands (/etc/sudoers.d/agent: apt, apt-get, dpkg,
  systemctl, service) and, for all other commands, sudo via a graphical
  ssh-askpass dialog (SUDO_ASKPASS, ssh-askpass-notify, sudo -A). Use when an
  agent needs root or sudo inside WSL, a password prompt, askpass, a graphical
  password dialog, cannot use an interactive TTY sudo, or when a
  Windows-native agent must run something inside this WSL machine. Not for
  Windows-side elevation (Windows has its own unrelated sudo.exe).
metadata:
  short-description: WSL host environment; sudo whitelist + ssh-askpass
---
# Local WSL execution environment

Host-specific notes for this machine's WSL environment, for agents running in
it or reaching into it. Do not treat these paths or wrappers as portable.

This skill covers two sudo paths: a NOPASSWD whitelist for routine commands,
and graphical ssh-askpass for everything else.

<!-- Maintainer note: write new sections as WSL-native bash. Do not add
per-topic wsl.exe wrappers — "Access from outside WSL" covers cross-side
invocation once. -->

## Access from outside WSL

Everything in this skill — paths like `/home/s/...`, wrappers, environment
variables — exists only **inside WSL**. How to consume it depends on where you run:

- **Already inside WSL**: run the snippets as written.
- **Windows-native agent** (e.g. Codex Desktop in PowerShell): wrap every
  command in a WSL login shell, e.g.:

  ```powershell
  wsl -e bash -lc 'SUDO_ASKPASS=/home/s/bin/ssh-askpass-notify sudo -A whoami'
  ```

  Do NOT run these commands in PowerShell: Windows 11 ships its own unrelated
  `sudo.exe`, and the WSL paths do not exist on the Windows side.

GUI output (password dialogs, notifications) appears on the Windows desktop via
WSLg no matter which side launched the command. Commands that wait for GUI
input need a generous timeout (about 2 minutes) — Windows-side harnesses often
default to much shorter.

## sudo without password (whitelisted commands)

`/etc/sudoers.d/agent` grants user `s` NOPASSWD for (any args allowed):

- `/usr/bin/apt`, `/usr/bin/apt-get`, `/usr/bin/dpkg`
- `/usr/bin/systemctl`, `/usr/sbin/service`

For these, run plain `sudo <command>` — no `-A`, no askpass, from either side
of WSL. For anything else, use the askpass flow below and expect a GUI dialog
on the user's side.

## sudo via ssh-askpass

Agent shells here have no sudo TTY. Do **not** run bare `sudo` for
non-whitelisted commands (it will fail with `a password is required` or
hang). Do **not** point `SUDO_ASKPASS` at `/usr/bin/ssh-askpass` directly if
the caller needs progress text — that binary prints nothing to the terminal
until `sudo` finishes.

**First, check the credential cache.** A successful GUI sudo is cached for
**2 hours**, shared by all of user `s`'s shells (`/etc/sudoers.d/timestamp`:
`timestamp_timeout=120`, `timestamp_type=global`). So after one successful
GUI sudo, run any later `sudo` directly — no pkill, no askpass, no password.
Only when a `sudo` call fails with `a password is required` (ticket expired)
do the full flow below.

**Before each askpass sudo, restart the WSLg compositor.** Agents cannot
tell whether the dialog actually painted, and weston wedges often on this
machine (long uptime, sleep/hibernate cycles, Fast Startup enabled), so do
not try to detect a stuck dialog — just restart first, and wait for it to
come back (a dialog opened too early shows GTK warnings / an empty warning
box; cancel and retry):

```bash
wsl.exe --system sh -lc "pkill -9 weston"
until wsl.exe --system sh -lc "pgrep -x weston >/dev/null"; do sleep 1; done
sleep 5
SUDO_ASKPASS=/home/s/bin/ssh-askpass-notify sudo -A <command>
```

The pkill works from inside WSL as written (via interop) and needs no sudo —
a bare in-WSL `pkill weston` does **not** work: weston lives in the WSLg
system distro's own PID namespace, invisible to user distros. If WSL interop
is broken too, run the same pkill from PowerShell instead.

Side effect: the pkill **closes all currently open WSL GUI apps** (browsers,
editors). If the user has WSL GUI apps open, warn them before running it.

Use the wrapper (calls `/usr/bin/ssh-askpass`, status on stderr, password only
on stdout):

```bash
SUDO_ASKPASS=/home/s/bin/ssh-askpass-notify sudo -A <command>
```

Example:

```bash
SUDO_ASKPASS=/home/s/bin/ssh-askpass-notify sudo -A whoami
```

`DISPLAY` is typically `:0` (WSLg). The user types the password in a GUI
window on their side. Give the command a long enough wait (about 2 minutes).

### Expected stderr

| Line | When |
|---|---|
| `ssh-askpass: waiting for password (graphical dialog)` | Dialog is about to open; wait for the user |
| `ssh-askpass: submitted, continuing` | User submitted; `sudo` continues |
| `ssh-askpass: cancelled or failed (exit …)` | Cancel or askpass error |

Wrong password: `sudo` retries askpass up to **3** times (a line per attempt),
then `sudo: 3 incorrect password attempts` and a non-zero exit.

### Rules

- Never log, echo, or paste the password.
- Do not `export SUDO_ASKPASS` into the user shell profile unless asked.
- Do not change `/usr/bin/ssh-askpass`.
- Do not edit `/etc/sudoers.d/agent` or `/etc/sudoers.d/timestamp` unless the
  user asks. Never whitelist editors or shells (`vim`, `sh`, `less`, …) —
  they escape to a full root shell.
- After a successful `sudo -A`, drop privileges in later commands unless root
  is still required.
