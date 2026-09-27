# Tailscale client on Ubuntu (AI Lab Headscale)

Tailscale puts your computer into the AI Lab private network, which is managed by **Headscale** at `https://headscale.ailab-sdu.com`. After joining you can reach lab hosts from anywhere (home, university Wi-Fi, mobile hotspot) by their Tailscale IP (`100.64.x.x`), but **only the hosts the Access Control List (ACL) allows for you**.

## 1. Get an auth key from the sysadmin

Ask the AI Lab sysadmin for a **pre-auth key**. You receive a command like:

```
tailscale up --login-server=https://headscale.ailab-sdu.com --authkey <YOUR_KEY>
```

* The key is personal. **Do not share it and do not commit it to git.**
* Keys can expire or be single-use. If it stops working, ask for a new one.

## 2. Install Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

The script adds the Tailscale apt repository, installs the package and starts the `tailscaled` service. Check it:

```bash
tailscale version
systemctl status tailscaled      # should be "active (running)", press q to exit
```

## 3. Connect to AI Lab Headscale

Run the command from the sysadmin (with `sudo`):

```bash
sudo tailscale up --login-server=https://headscale.ailab-sdu.com --authkey <YOUR_KEY>
```

If you were logged into another Tailscale account before, add `--force-reauth`, or run `sudo tailscale logout` first.

No output means success. Check the connection:

```bash
tailscale status
```

Example output (your IP and the hosts you see will differ):

```
100.64.0.7    my-laptop       my-user   linux    -
100.64.0.12   lab-ubuntu      lab       linux    -
100.64.0.15   LAB-PC   lab       windows  -
```

Your own Tailscale IP:

```bash
tailscale ip -4
```

Now you are connected to AI Lab's Headscale.

## 4. Connect to AI Lab hosts

Use the Tailscale IP of the host from `tailscale status`:

```bash
tailscale ping 100.64.0.12        # checks that the host is reachable over Tailscale
ssh <user>@100.64.0.12
```

If MagicDNS is enabled on the Headscale server (ask the sysadmin), the host name works too:

```bash
ssh <user>@lab-ubuntu
```

Put the host into your SSH config file to use it with `ssh` and VS Code (see `client/ssh/ubuntu/README.md`, steps 5 and 6):

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
| Temporarily disconnect | `sudo tailscale down` |
| Reconnect (same account) | `sudo tailscale up --login-server=https://headscale.ailab-sdu.com` |
| Leave the network completely | `sudo tailscale logout` |

## Troubleshooting

| Problem | Fix |
|---|---|
| `tailscale: command not found` | Run the install script again (step 2). |
| `failed to connect to local tailscaled` | Start the service: `sudo systemctl enable --now tailscaled`. |
| `invalid key` / `authkey expired` | The key is expired or already used. Ask the sysadmin for a new one. |
| `can't change --login-server without --force-reauth` | Add `--force-reauth` to the `tailscale up` command. |
| `Access denied: ... requires root` | Use `sudo` before `tailscale up` / `down` / `logout`. |
| `tailscale ping` works but `ssh` times out | The ACL does not allow SSH (port 22) to this host, or the SSH server is not running. |
| Host is missing in `tailscale status` | The ACL hides it from you, or the host is offline. Ask the sysadmin. |
