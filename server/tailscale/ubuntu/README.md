# Tailscale on an Ubuntu server (AI Lab Headscale)

Joins this Ubuntu machine to the AI Lab private network (Headscale at `https://headscale.ailab-sdu.com`) so users can reach its SSH server from anywhere through its Tailscale IP (`100.64.x.x`). Who may connect is decided by the sysadmin's **Access Control List (ACL)**.

Set up the SSH server first: `server/ssh/ubuntu/README.md`.

## 1. Get an auth key from the sysadmin

Ask the AI Lab sysadmin for a **pre-auth key for a server**. You receive a command like:

```
tailscale up --login-server=https://headscale.ailab-sdu.com --authkey <YOUR_KEY>
```

Do not share the key and do not commit it to git.

## 2. Install Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo systemctl enable --now tailscaled      # start now and after every reboot
```

## 3. Connect to AI Lab Headscale

```bash
sudo tailscale up --login-server=https://headscale.ailab-sdu.com --authkey <YOUR_KEY> --hostname=$(hostname)
```

`--hostname` is the name of this machine in the network (what users see in `tailscale status`).

Check it:

```bash
tailscale status
tailscale ip -4          # give this IP to the users / sysadmin
```

The `tailscaled` service reconnects automatically after reboot, nobody needs to be logged in.

## 4. Allow SSH through the firewall

If UFW is active, SSH must be allowed (already done in `server/ssh/ubuntu/README.md`):

```bash
sudo ufw allow OpenSSH
```

To allow SSH **only** through Tailscale (not from the LAN/internet):

```bash
sudo ufw allow in on tailscale0 to any port 22 proto tcp
sudo ufw delete allow OpenSSH
```

## 5. Test from a client

From another computer that is also in AI Lab Headscale (see `client/tailscale/`):

```bash
tailscale ping <hostname of this server>
ssh <user>@<Tailscale IP of this server>
```

If `ping` works but `ssh` times out, check step 4 and ask the sysadmin whether the ACL allows port 22 to this host.

## Useful commands

| Action | Command |
|---|---|
| Show connection state and peers | `tailscale status` |
| This machine's Tailscale IP | `tailscale ip -4` |
| Service logs | `sudo journalctl -u tailscaled -n 50` |
| Temporarily disconnect | `sudo tailscale down` |
| Leave the network completely | `sudo tailscale logout` |

> Ask the sysadmin to **disable key expiry** for this node in Headscale, otherwise the server may drop out of the network when its key expires.
