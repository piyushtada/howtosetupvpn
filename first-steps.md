# First steps on a new VPS

Covers the initial setup of a fresh VPS: installing basic tools and configuring
password-less SSH login.

> Throughout this guide, replace the placeholders with your own values:
> - `vps-name` / `vpsname` — a short name for the server (e.g. `web01`).
> - `203.0.113.10` — the server's IP address.
> - `my-vps` — the alias you want to use in `~/.ssh/config`.

## 1. Install basic tools

```bash
sudo apt update && sudo apt install -y curl wget
```

## 2. Set up password-less SSH login

### Generate an SSH key

Run this on your **local** machine. Choose a unique filename per server so you can
manage multiple VPS hosts independently.

```bash
ssh-keygen -t ed25519 -C "vps-name" -f ~/.ssh/id_ed25519_vpsname
```

- `-t ed25519` — the recommended, modern key type.
- `-C "vps-name"` — a comment (label) to help identify the key later.
- `-f ~/.ssh/id_ed25519_vpsname` — the output file path for this key pair.
- You will be prompted for an optional passphrase. A passphrase is recommended;
  you can cache it with `ssh-agent` so you only type it once per session.

### Copy the public key to the server

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_vpsname.pub root@203.0.113.10
```

This authenticates once with your password and appends the public key to
`~/.ssh/authorized_keys` on the server. If password authentication is already
disabled, append the contents of `~/.ssh/id_ed25519_vpsname.pub` to
`~/.ssh/authorized_keys` on the server manually instead.

### Configure the SSH client

Add the following to `~/.ssh/config` on your **local** machine so you can connect
with a short alias (`ssh my-vps`):

```sshconfig
Host my-vps
    HostName 203.0.113.10
    User root
    Port 22
    IdentityFile ~/.ssh/id_ed25519_vpsname
    IdentitiesOnly yes
    ServerAliveInterval 60
```

> **Note:** `IdentityFile` must match the key you generated in the previous step.
> The original draft generated `~/.ssh/id_ed25519_vpsname` but pointed
> `IdentityFile` at `~/.ssh/id_ed25519`, which would fail to authenticate.

Once saved, connect with:

```bash
ssh my-vps
```

## 3. Recommended hardening (optional)

After confirming that key-based login works:

- Disable password authentication in `/etc/ssh/sshd_config` by setting
  `PasswordAuthentication no`, then restart the service with
  `sudo systemctl restart ssh`.
- Consider disabling direct `root` login (`PermitRootLogin no`) and using a
  regular user with `sudo` instead.

> Keep your current SSH session open while testing changes so you don't lock
> yourself out.
