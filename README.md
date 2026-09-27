<div align="center">

<img src="assets/icons/ssh.svg" width="64" alt="SSH">&nbsp;&nbsp;
<img src="assets/icons/tailscale.svg" width="64" alt="Tailscale">&nbsp;&nbsp;
<img src="assets/icons/vscode.svg" width="64" alt="VS Code">

# AI Lab Remote Access Configuration

Connect to AI Lab computers over **SSH**, from the lab or from anywhere through the AI Lab **Tailscale** network<br>
(Headscale at `https://headscale.ailab-sdu.com`), and work on them in **VS Code**.

</div>

---

## How it works

```mermaid
flowchart LR
    C["💻 Your computer<br/>(client)"] -- "Tailscale<br/>100.64.x.x" --> H(("Headscale<br/>AI Lab network"))
    H -- "allowed by ACL" --> S["🖥️ Lab PC<br/>(server, OpenSSH)"]
    C -. "ssh / VS Code Remote-SSH" .-> S
```

* **Server** is the lab computer you connect **to**.
* **Client** is your own computer you connect **from**.

## Guides

| | <img src="assets/icons/windows.svg" width="22" align="center"> Windows | <img src="assets/icons/ubuntu.svg" width="22" align="center"> Ubuntu |
|---|---|---|
| <img src="assets/icons/ssh.svg" width="22" align="center"> **Server: SSH**<br>install and start OpenSSH Server | [server/ssh/windows](server/ssh/windows/README.md) | [server/ssh/ubuntu](server/ssh/ubuntu/README.md) |
| <img src="assets/icons/tailscale.svg" width="22" align="center"> **Server: Tailscale**<br>join the lab network | [server/tailscale/windows](server/tailscale/windows/README.md) | [server/tailscale/ubuntu](server/tailscale/ubuntu/README.md) |
| <img src="assets/icons/tailscale.svg" width="22" align="center"> **Client: Tailscale**<br>join the lab network | [client/tailscale/windows](client/tailscale/windows/README.md) | [client/tailscale/ubuntu](client/tailscale/ubuntu/README.md) |
| <img src="assets/icons/ssh.svg" width="22" align="center"> **Client: SSH**<br>keys and config file | [client/ssh/windows](client/ssh/windows/README.md) | [client/ssh/ubuntu](client/ssh/ubuntu/README.md) |
| <img src="assets/icons/vscode.svg" width="22" align="center"> **Client: VS Code**<br>Remote-SSH connection | [VS Code on Windows](client/ssh/windows/README.md#6-connect-with-vs-code-remote---ssh) | [VS Code on Ubuntu](client/ssh/ubuntu/README.md#6-connect-with-vs-code-remote---ssh) |

## Recommended order

1. <img src="assets/icons/ssh.svg" width="18" align="center"> **On the server:** set up SSH,
   then <img src="assets/icons/tailscale.svg" width="18" align="center"> join Tailscale.
2. <img src="assets/icons/tailscale.svg" width="18" align="center"> **On the client:** join Tailscale with the auth key from the sysadmin.
3. <img src="assets/icons/ssh.svg" width="18" align="center"> **On the client:** set up SSH: key and config file.
4. <img src="assets/icons/vscode.svg" width="18" align="center"> **Connect:** `ssh <host>` in a terminal, or **Remote-SSH: Connect to Host...** in VS Code.

> [!NOTE]
> Access to each host is controlled by the sysadmin's **Access Control List (ACL)**. If a host is not reachable, ask the sysadmin.
