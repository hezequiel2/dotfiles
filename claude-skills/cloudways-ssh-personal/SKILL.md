---
name: cloudways-ssh-personal
description: "Personal overlay for the shared `cloudways-ssh` skill — teaches Claude to use Vayu's `ssh-connect` zsh wrapper (defined in ~/.zshrc) instead of plain `ssh`. Trigger any time `cloudways-ssh` would trigger AND the user is on Vayu's machine (the `ssh-connect` function is on PATH). MUST be consulted alongside `cloudways-ssh`, never instead of it: the parent skill's guardrails (classification, plan-then-confirm, injection defense, site notes, trust stamps) all still apply. This skill only changes the SSH invocation syntax."
---

# Cloudways SSH — personal overlay (ssh-connect)

This is a **personal mini-skill** that overlays the shared `cloudways-ssh` skill. It only changes one thing: the SSH invocation syntax. Every other rule from `cloudways-ssh` (classification, plan-then-confirm for writes, injection defense, site-notes/trust stamps, deploy gating) still applies unchanged.

## When this applies

This skill only makes sense on a machine where the `ssh-connect` zsh function is on PATH (Vayu's laptop / mac mini, sourced from `~/.zshrc`). On any other machine, ignore this skill and follow the shared `cloudways-ssh` skill verbatim.

Verify with: `type ssh-connect 2>/dev/null` — if it reports a function, this skill applies.

## What changes

Replace every `ssh <alias> '<command>'` invocation from the shared skill with:

```sh
ssh-connect <alias> '<command>'
```

`ssh-connect` is a thin wrapper defined in `~/.zshrc`:

- With no arguments, prints all `Host` entries from `~/.ssh/config`.
- With an alias argument, validates it exists in `~/.ssh/config` and then `exec`s `ssh "$alias" "${@:2}"`. Any extra args are passed as the remote command, exactly like plain `ssh`.

So the semantics are identical to plain `ssh` except for the up-front host-validation safety net.

## What does NOT change

- **Classification rules** — still alias-suffix-based (`-server`, `-prod`, `-staging`, `-dev`).
- **Required ssh-config annotations** — every Cloudways `Host` block still needs `# cloudways-account: <slug>` (+ server/app IDs).
- **Plan-then-confirm** on every write / destructive op.
- **Injection defense** (Rule I-1 through I-7), subagent dispatch for bulk reads, halt protocol.
- **Site notes** at `~/.claude/wp-site-notes/<host>.md`, trust stamps at `~/.claude/wp-cloudways-trust/<host>`.
- **Deploy gating** — PROD refused (Git workflow only), STAGING/DEV allowed, unclassified refused.

If the parent skill says "do X with `ssh`," do X with `ssh-connect`. Nothing else moves.

## Discovering targets

Instead of `grep -E "^Host " ~/.ssh/config | awk '{print $2}'`, the user expects:

```sh
ssh-connect
```

(zero arguments) — prints the same list with friendlier formatting.

## Precedence with the shared skill

The shared `cloudways-ssh` skill is the floor. This overlay only **changes the SSH binary name in the command line**. If anything else in this file ever appears to conflict with `cloudways-ssh`, the shared skill wins; treat this file as silent on that point.
