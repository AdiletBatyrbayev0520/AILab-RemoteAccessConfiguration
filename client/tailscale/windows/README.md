<p>
<img src="../../../assets/icons/tailscale.png" width="48" alt="tailscale">&nbsp;
<img src="../../../assets/icons/windows.svg" width="48" alt="windows">
</p>

# Tailscale client on Windows (AI Lab Headscale)

[← Back to all guides](../../../README.md)

Tailscale puts your computer into the AI Lab private network, which is managed by **Headscale** at `https://headscale.ailab-sdu.com`. After joining you can reach lab hosts from anywhere (home, university Wi-Fi, mobile hotspot) by their Tailscale IP (`100.64.x.x`), but **only the hosts the Access Control List (ACL) allows for you**.

## 1. Get an auth key from the sysadmin

Ask the AI Lab sysadmin for a **pre-auth key**. You receive a command like:

```
tailscale up --login-server=https://headscale.ailab-sdu.com --authkey <YOUR_KEY>
```

* The key is personal. **Do not share it and do not commit it to git.**
* Keys can expire or be single-use. If it stops working, ask for a new one.

## 2. Install Tailscale

1. Download the Windows installer from <https://tailscale.com/download/windows>.
2. Run it and finish the installation. A Tailscale icon appears in the system tray (near the clock).
3. **Do not** click *Log in* in the tray menu. That logs into tailscale.com, not into the AI Lab server.

Check that the command-line tool works. Open **PowerShell as Administrator** and run:

```powershell
tailscale version
```

If it is not found, close and reopen PowerShell, or use the full path `& "C:\Program Files\Tailscale\tailscale.exe" version`.

## 3. Connect to AI Lab Headscale

In **PowerShell as Administrator**, run the command from the sysadmin:

```powershell
tailscale up --login-server=https://headscale.ailab-sdu.com --authkey <YOUR_KEY>
```

If you were logged into another Tailscale account before, add `--force-reauth`, or run `tailscale logout` first.

No output means success. Check the connection:

```powershell
tailscale status
```

Example output (your IP and the hosts you see will differ):

```
100.64.0.7    my-laptop       my-user   windows  -
100.64.0.12   lab-ubuntu      lab       linux    -
100.64.0.15   LAB-PC   lab       windows  -
```

Your own Tailscale IP:

```powershell
tailscale ip -4
```

Now you are connected to AI Lab's Headscale.

## 4. Connect to AI Lab hosts

<img src="../../../assets/icons/ssh.svg" width="40" alt="SSH">&nbsp;<img src="../../../assets/icons/vscode.svg" width="40" alt="VS Code">

Use the Tailscale IP of the host from `tailscale status`:

```powershell
tailscale ping 100.64.0.12        # checks that the host is reachable over Tailscale
ssh <user>@100.64.0.12
```

If MagicDNS is enabled on the Headscale server (ask the sysadmin), the host name works too:

```powershell
ssh <user>@lab-ubuntu
```

Put the host into your SSH config file to use it with `ssh` and VS Code (see `client/ssh/windows/README.md`, steps 5 and 6):

```
Host lab-ubuntu
    HostName 100.64.0.12
    User <user>
    IdentityFile ~/.ssh/id_ed25519
```

## Access Control List (ACL)

Being in the network does **not** mean you can reach every host. The sysadmin's ACL decides which hosts and ports each user may connect to.

* A host you have no access to may not appear in `tailscale status`, and `ssh` to it ends with **Connection timed out**.
* If you need access to a host, ask the sysadmin to add you to the ACL.

## Useful commands

| Action | Command |
|---|---|
| Show connection state and peers | `tailscale status` |
| Your Tailscale IP | `tailscale ip -4` |
| Check a host is reachable | `tailscale ping <IP or name>` |
| Temporarily disconnect | `tailscale down` |
| Reconnect (same account) | `tailscale up --login-server=https://headscale.ailab-sdu.com` |
| Leave the network completely | `tailscale logout` |

## Troubleshooting

| Problem | Fix |
|---|---|
| `tailscale` is not recognized | Reopen PowerShell, or use `& "C:\Program Files\Tailscale\tailscale.exe"`. |
| `invalid key` / `authkey expired` | The key is expired or already used. Ask the sysadmin for a new one. |
| `can't change --login-server without --force-reauth` | Add `--force-reauth` to the `tailscale up` command. |
| `tailscale ping` works but `ssh` times out | The ACL does not allow SSH (port 22) to this host, or the SSH server is not running. |
| Host is missing in `tailscale status` | The ACL hides it from you, or the host is offline. Ask the sysadmin. |
