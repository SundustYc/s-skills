---
name: local-wsl-env
description: >-
  This WSL machine's host-specific execution environment. Currently documents
  how to request sudo via a graphical ssh-askpass dialog (SUDO_ASKPASS,
  ssh-askpass-notify, sudo -A). Use when an agent needs root, sudo, a password
  prompt, askpass, graphical password dialog, or cannot use an interactive TTY sudo.
metadata:
  short-description: WSL host environment; sudo via ssh-askpass
---
# Local WSL execution environment

Host-specific notes for agents running **on this WSL machine**. Do not treat
these paths or wrappers as portable.

Currently this skill only covers graphical `sudo` via ssh-askpass.

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
