---
name: local-wsl-env
description: >-
  This WSL machine's host-specific execution environment; all commands and
  paths exist only inside WSL. Currently documents how to request sudo via a
  graphical ssh-askpass dialog (SUDO_ASKPASS, ssh-askpass-notify, sudo -A).
  Use when an agent needs root or sudo inside WSL, a password prompt, askpass,
  a graphical password dialog, cannot use an interactive TTY sudo, or when a
  Windows-native agent must run something inside this WSL machine. Not for
  Windows-side elevation (Windows has its own unrelated sudo.exe).
metadata:
  short-description: WSL host environment; sudo via ssh-askpass
---
# Local WSL execution environment

Host-specific notes for this machine's WSL environment, for agents running in
it or reaching into it. Do not treat these paths or wrappers as portable.

Currently this skill only covers graphical `sudo` via ssh-askpass.

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

## sudo via ssh-askpass

Agent shells here have no sudo TTY. Do **not** run bare `sudo` (it will fail
with `a password is required` or hang). Do **not** point `SUDO_ASKPASS` at
`/usr/bin/ssh-askpass` directly if the caller needs progress text — that binary
prints nothing to the terminal until `sudo` finishes.

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

Success caches credentials for a short period. A later `sudo -A` in that
window may skip the dialog and produce no askpass lines.

### Rules

- Never log, echo, or paste the password.
- Do not `export SUDO_ASKPASS` into the user shell profile unless asked.
- Do not change `/usr/bin/ssh-askpass` or `sudoers` for this flow.
- After a successful `sudo -A`, drop privileges in later commands unless root
  is still required.
