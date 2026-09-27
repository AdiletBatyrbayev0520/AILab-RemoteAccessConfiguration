# AI Lab Remote Access Configuration

Guides for connecting to AI Lab computers over SSH, locally or from anywhere through the AI Lab Tailscale network (Headscale at `https://headscale.ailab-sdu.com`).

* **Server** is the computer you connect **to** (a lab PC).
* **Client** is the computer you connect **from** (your laptop).

## Guides

| | Windows | Ubuntu |
|---|---|---|
| **Server: SSH** (install and start OpenSSH Server) | [server/ssh/windows](server/ssh/windows/README.md) | [server/ssh/ubuntu](server/ssh/ubuntu/README.md) |
| **Server: Tailscale** (join the lab network) | [server/tailscale/windows](server/tailscale/windows/README.md) | [server/tailscale/ubuntu](server/tailscale/ubuntu/README.md) |
| **Client: SSH** (keys, config file, VS Code) | [client/ssh/windows](client/ssh/windows/README.md) | [client/ssh/ubuntu](client/ssh/ubuntu/README.md) |
| **Client: Tailscale** (join the lab network) | [client/tailscale/windows](client/tailscale/windows/README.md) | [client/tailscale/ubuntu](client/tailscale/ubuntu/README.md) |

## Recommended order

1. **On the server:** set up SSH, then join Tailscale.
2. **On the client:** join Tailscale with the auth key from the sysadmin, then set up SSH (key, config file, VS Code).
3. Connect: `ssh <user>@<Tailscale IP of the server>`, or with the `Host` name from your config file.

Access to each host is controlled by the sysadmin's Access Control List (ACL). If a host is not reachable, ask the sysadmin.
