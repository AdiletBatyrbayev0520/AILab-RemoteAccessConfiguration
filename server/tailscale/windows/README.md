# Tailscale on a Windows server (AI Lab Headscale)

Joins this Windows PC to the AI Lab private network (Headscale at `https://headscale.ailab-sdu.com`) so users can reach its SSH server from anywhere through its Tailscale IP (`100.64.x.x`). Who may connect is decided by the sysadmin's **Access Control List (ACL)**.

Set up the SSH server first: `server/ssh/windows/README.md`.

## 1. Get an auth key from the sysadmin

Ask the AI Lab sysadmin for a **pre-auth key for a server**. You receive a command like:

```
tailscale up --login-server=https://headscale.ailab-sdu.com --authkey <YOUR_KEY>
```

Do not share the key and do not commit it to git.

## 2. Install Tailscale

1. Download the Windows installer from <https://tailscale.com/download/windows>.
2. Run it and finish the installation. A Tailscale icon appears in the system tray.
3. **Do not** click *Log in* in the tray menu. That logs into tailscale.com, not into the AI Lab server.

## 3. Connect to AI Lab Headscale

Open **PowerShell as Administrator** and run:

```powershell
tailscale up --login-server=https://headscale.ailab-sdu.com --authkey <YOUR_KEY> --hostname=LAB-PC --unattended
```

| Flag | Why |
|---|---|
| `--hostname` | Name of this PC in the network (what users see in `tailscale status`). Use the PC name. |
| `--unattended` | Keeps Tailscale connected when nobody is signed into Windows and after reboot. **Required for a server.** |

Check it:

```powershell
tailscale status
tailscale ip -4          # give this IP to the users / sysadmin
```

## 4. Allow SSH through the Tailscale network adapter

The OpenSSH firewall rule is enabled for the **Private** profile only. Check which profile the Tailscale adapter has:

```powershell
Get-NetConnectionProfile | Format-Table InterfaceAlias, NetworkCategory
```

If `Tailscale` shows `Public`, pick one of these:

```powershell
# a) make the Tailscale network Private
Set-NetConnectionProfile -InterfaceAlias Tailscale -NetworkCategory Private

# b) or allow the SSH rule on every profile
Set-NetFirewallRule -Name OpenSSH-Server-In-TCP -Profile Any
```

## 5. Test from a client

From another computer that is also in AI Lab Headscale (see `client/tailscale/`):

```powershell
tailscale ping LAB-PC
ssh <user>@<Tailscale IP of this PC>
```

If `ping` works but `ssh` times out, check step 4 and ask the sysadmin whether the ACL allows port 22 to this host.

## Useful commands

| Action | Command |
|---|---|
| Show connection state and peers | `tailscale status` |
| This PC's Tailscale IP | `tailscale ip -4` |
| Temporarily disconnect | `tailscale down` |
| Leave the network completely | `tailscale logout` |

> Ask the sysadmin to **disable key expiry** for this node in Headscale, otherwise the server may drop out of the network when its key expires.
