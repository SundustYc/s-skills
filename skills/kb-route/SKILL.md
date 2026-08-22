---
name: kb-route
description: >-
  Route Obsidian knowledge-base lookups and reusable-knowledge capture to the
  user's vault after reading its AGENT.md. Use when an agent needs the vault
  path, local conventions, knowledge-base / 知识库 / 笔记库 routing, or to propose
  writing reusable results into the vault. Do not use for Obsidian-flavored
  Markdown, CLI usage, Bases, or Canvas files unless vault location or
  write-approval policy is also needed; use obsidian-markdown, obsidian-cli,
  obsidian-bases, or json-canvas for those.
metadata:
  short-description: Route work to the Obsidian knowledge base
---

# Obsidian Knowledge Base Route

Use this skill as the location and policy entry point for the user's Obsidian knowledge base.

## Vault Location

- WSL path: `/mnt/d/notes/obsidian`
- Windows path: `D:\Notes\obsidian`

The WSL path is the canonical path for shell commands in this environment. Do not assume that
the current project directory is the vault.

## Before Vault Work

Before searching, interpreting, or changing vault content, read the vault root's
`/mnt/d/notes/obsidian/AGENT.md`. It contains the vault-specific structure, conventions, and
operating rules. If that file is missing or cannot be read, tell the user before making
vault-specific assumptions or edits.

This check is especially relevant when:

- looking up related knowledge before starting a complex task;
- checking how a topic, folder, tag, property, or note should be organized; or
- deciding where reusable results from the current task belong.

## Use Existing Obsidian Skills

Load and follow the relevant installed Obsidian skill in addition to this route skill:

- `obsidian-cli` for searching, reading, creating, appending, or otherwise managing notes through
  the Obsidian CLI (including its requirement that Obsidian be running);
- `obsidian-markdown` for Obsidian-flavored Markdown, wikilinks, embeds, callouts, and properties;
- `obsidian-bases` for `.base` files; and
- `json-canvas` for `.canvas` files.

Use only the skills relevant to the requested artifact or operation; this skill does not replace
their detailed format or tool instructions.

If Obsidian is not running, do not treat the CLI as available. Read and search the vault on the
WSL filesystem path, or ask the user to open Obsidian when a CLI-only operation is required.

## Reusable Knowledge and Writes

When work produces knowledge that may be useful later, first explain to the user what should be
captured and the proposed vault path or note. Ask whether they want it written to the knowledge
base. Write only after the user confirms, then follow `AGENT.md` and the relevant Obsidian skill.

A request to inspect or search the vault does not by itself authorize writing to it. Keep temporary
notes, drafts, and task-specific output outside the vault unless the user explicitly approves a
vault write. Do not copy vault notes into the current project; the vault is the knowledge store,
and the project is separate working files.
