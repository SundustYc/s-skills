---
name: kimi-code
description: >-
  Invoke the Kimi Code CLI (`kimi`) so another agent can start, resume, or
  script a Kimi coding session. Use when the user or agent needs to call Kimi
  Code, kimi-code, kimi CLI, Moonshot, k3, kimi-for-coding, `kimi -p`,
  `kimi acp`, `kimi web`, or 调用/启动 kimi.
metadata:
  short-description: How other agents should invoke the Kimi Code CLI
---
# Kimi Code CLI

Use this skill to call **Kimi Code** from a shell. The command is `kimi`.
Do not hardcode a machine-specific absolute path.

Official docs: https://moonshotai.github.io/kimi-code/

## Resolve the binary

```bash
command -v kimi || export PATH="$HOME/.kimi-code/bin:$PATH"
command -v kimi
kimi --version
```

If `kimi` is still missing, tell the user the CLI is not on `PATH` and stop.
Typical install location is `$HOME/.kimi-code/bin` (added from `~/.bashrc`).
Config lives in `$HOME/.kimi-code/` (`config.toml`, `tui.toml`, `region`).
Do not print `api_key` values from `config.toml`.

```bash
kimi doctor          # validate config.toml / tui.toml
kimi provider list   # providers and default model
```

## Which invocation to use

| Situation | Command |
|---|---|
| Agent / script / no TTY | `kimi -p "..."` |
| User is in an interactive terminal | `kimi` |
| Continue last session in this cwd | `kimi -c` |
| Resume a known session | `kimi -S <sessionId>` |
| IDE ACP stdio | `kimi acp` |
| Browser UI | `kimi web --no-open` |

Always `cd` to the target project first. The workspace is the current directory.

**Agent default:** use `-p`. Do not start the TUI (`kimi` with no args) from a
non-interactive agent shell.

## Commands

### One-shot (preferred for agents)

```bash
kimi -p "PROMPT"
kimi -p "PROMPT" --output-format text
kimi -p "PROMPT" --output-format stream-json
kimi -m kimi-code/k3 -p "PROMPT"
kimi --skills-dir ./skills -p "PROMPT"
kimi --add-dir ../other-project -p "PROMPT"
```

`-p` cannot be combined with `-y`, `--auto`, or `--plan`. Tool permission then
follows `default_permission_mode` in `$HOME/.kimi-code/config.toml`.

### Interactive (real TTY only)

```bash
kimi                         # TUI in cwd
kimi -m kimi-code/k3
kimi -y                      # Ask When Needed
kimi --auto                  # Never Ask
kimi --plan                  # plan first
kimi --skills-dir ./skills
kimi --add-dir ../other-project
kimi --agent <name>
kimi --agent-file path/to/agent.md
```

Inside the TUI, type `/help` for slash commands.

### Sessions

```bash
kimi session list
kimi -c                      # continue previous session for this cwd
kimi -S                      # picker
kimi -S session_<id>
kimi export [sessionId]
kimi fork [sessionId]
kimi vis [sessionId]
```

`-c` and `-S` are mutually exclusive.

### Auth and providers

```bash
kimi login --region mainland-cn    # kimi.com
kimi login --region global         # kimi.ai
kimi provider list
kimi provider catalog
kimi provider add <url>            # import from a custom api.json registry
kimi provider remove <providerId>
```

Pick region from `$HOME/.kimi-code/region` when present. Do not re-login unless
`provider list` / a 401 shows the session is unauthenticated.

### Other

```bash
kimi web                       # local UI, default 127.0.0.1:58627
kimi web --no-open
kimi web --port 58627
kimi acp                       # Agent Client Protocol over stdio
kimi acp --login --region mainland-cn
kimi doctor
kimi upgrade                   # alias: kimi update
kimi migrate                   # from legacy kimi-cli
```

## Models

Pass aliases with `-m`. Confirm with `kimi provider list`. Common managed aliases:

| Alias | Notes |
|---|---|
| `kimi-code/k3` | default on a stock managed install |
| `kimi-code/k3-256k` | K3, 256k context |
| `kimi-code/kimi-for-coding` | K2.7 Coding |
| `kimi-code/kimi-for-coding-highspeed` | K2.7 highspeed |

Override per invocation; do not edit `config.toml` unless the user asks.

## Flag conflicts

- `-c` and `-S` cannot be combined
- `-y` and `--auto` cannot be combined
- `-p` cannot be combined with `-y`, `--auto`, or `--plan`
- `--output-format` requires `-p`
- `--agent` / `--agent-file` cannot be combined with `-S` / `-c`

## Skills

`--skills-dir` replaces auto-discovered user/project skill dirs (repeatable).
To load this repo's skills from the repo root:

```bash
kimi --skills-dir ./skills -p "PROMPT"
```

## Failure handling

1. `command not found` → prepend `$HOME/.kimi-code/bin` once; if still missing, stop.
2. Unauthenticated / 401 → `kimi login --region …` (needs user/browser).
3. `kimi doctor` fails → report the file and error; do not invent config.
4. Hook errors on Bash tools → `$HOME/.kimi-code/config.toml` may reference
   `$HOME/.kimi-code/hooks/`; tell the user if that script is missing.
5. Never dump API keys, device IDs, or bearer tokens from config or `kimi web`.
