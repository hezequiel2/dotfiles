---
name: wordpress-ssh-personal
description: "Personal overlay for the shared `wordpress-ssh` skill — teaches Claude to use Vayu's `ssh-connect` zsh wrapper (defined in ~/.zshrc) instead of plain `ssh`. Trigger any time `wordpress-ssh` would trigger AND the user is on Vayu's machine (the `ssh-connect` function is on PATH) — i.e. any SSH work on a WordPress site hosted on Cloudways, Hostinger, or any host annotated in ~/.ssh/config: wp-cli, wp-config, debug.log, DB queries, log reads, file edits. MUST be consulted alongside `wordpress-ssh`, never instead of it: the parent skill's guardrails (provider resolution, classification, explicit site root, plan-then-confirm, injection defense, site notes, trust stamps) all still apply. This skill only changes the SSH invocation syntax."
---

# WordPress SSH — personal overlay (ssh-connect)

This is a **personal mini-skill** that overlays the shared `wordpress-ssh` skill. It only changes one thing: the SSH invocation syntax. Every other rule from `wordpress-ssh` (provider resolution, classification, explicit site root, plan-then-confirm for writes, injection defense, site-notes/trust stamps, deploy gating) still applies unchanged.

## When this applies

This skill only makes sense on a machine where the `ssh-connect` zsh function is on PATH (Vayu's laptop / mac mini, sourced from `~/.zshrc`). On any other machine, ignore this skill and follow the shared `wordpress-ssh` skill verbatim.

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

**Provider-agnostic.** `ssh-connect` passes the alias straight to `ssh`, so everything encoded in the `Host` block still applies — including non-standard ports (`Port 65002` on Hostinger shared) and `IdentitiesOnly yes`. This is exactly why the parent skill's Golden Rule 10 says to connect by alias and never by raw host/IP/port: the wrapper never needs provider-specific flags.

Examples:

```sh
# Cloudways (site scope) — site root is public_html
ssh-connect ananda-org-prod 'cd public_html && wp core version'

# Hostinger (account scope) — site root MUST be explicit per parent Golden Rule 4
ssh-connect ananda-india-server 'cd ~/domains/anandaindia.org/public_html && wp core version'
```

## What does NOT change

- **Provider resolution** — still mandatory before the first command, still driven by the `# cloudways-account:` / `# hostinger-account:` annotation, still requires reading the matching `references/providers/<provider>.md`.
- **Classification rules** — still alias-suffix-based (`-server`, `-prod`, `-staging`, `-dev`).
- **Explicit site root** — Golden Rule 4. `ssh-connect` validates the *host*, never the *path*. On account-scope hosts it offers zero protection against targeting the wrong site, so the site root still goes in every command and still gets verified before writes.
- **Required ssh-config annotations** — every `Host` block still needs its provider annotation (plus server/app IDs on Cloudways).
- **Plan-then-confirm** on every write / destructive op.
- **Injection defense** (Rule I-1 through I-7), subagent dispatch for bulk reads, halt protocol.
- **Site notes** at `~/.claude/wp-site-notes/<host>.md`, trust stamps at `~/.claude/wp-ssh-trust/<host>`.
- **Deploy gating** — PROD refused (Git workflow only), STAGING/DEV allowed, unclassified refused, account-scope refused.

If the parent skill says "do X with `ssh`," do X with `ssh-connect`. Nothing else moves.

## Discovering targets

Instead of `grep -E "^Host " ~/.ssh/config | awk '{print $2}'`, the user expects:

```sh
ssh-connect
```

(zero arguments) — prints the same list with friendlier formatting.

Note that this lists aliases only, not their provider annotations. To resolve a host's provider you still read `~/.ssh/config` directly:

```sh
grep -B 8 "^Host <alias>$" ~/.ssh/config
```

## Precedence with the shared skill

The shared `wordpress-ssh` skill is the floor. This overlay only **changes the SSH binary name in the command line**. If anything else in this file ever appears to conflict with `wordpress-ssh`, the shared skill wins; treat this file as silent on that point.
