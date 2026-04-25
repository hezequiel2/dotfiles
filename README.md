# dotfiles

Personal dotfiles, synced between Vayu's Mac Mini and Linux Manjaro machine.

## What's here

- `ssh_config` — canonical `~/.ssh/config`, symlinked into `~/.ssh/` on each machine.

## Onboard a new machine

```sh
git clone git@github.com:hezequiel2/dotfiles.git ~/dotfiles
mv ~/.ssh/config ~/.ssh/config.bak.$(date +%Y%m%d) 2>/dev/null || true
ln -s ~/dotfiles/ssh_config ~/.ssh/config
chmod 600 ~/dotfiles/ssh_config
```

Verify:

```sh
ssh -G <some-host>   # should resolve as expected
```

## Add or edit a host

1. Edit `~/dotfiles/ssh_config` on whichever machine you're on.
2. `cd ~/dotfiles && git add ssh_config && git commit -m "ssh: <what changed>" && git push`
3. On the other machine: `cd ~/dotfiles && git pull`.

Never edit `~/.ssh/config` directly — it's a symlink, but more importantly, ad-hoc edits skip git and the two machines drift again.

## Rules

- Never commit private keys (`id_*`, `*.pem`). Only `ssh_config`.
- Repo is private — but still avoid putting secrets in host entries.
