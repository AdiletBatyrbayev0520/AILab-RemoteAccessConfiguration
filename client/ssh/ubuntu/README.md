# OpenSSH Client on Ubuntu

The client is the computer you connect **from**. The server must already be set up (see `server/ssh/`).

## 1. Install OpenSSH Client

The client is installed on Ubuntu by default. Check it:

```bash
ssh -V
```

Expected output (version may differ):

```
OpenSSH_9.6p1 Ubuntu-3ubuntu13, OpenSSL 3.0.13 30 Jan 2024
```

If you get `ssh: command not found`, install it:

```bash
sudo apt update
sudo apt install -y openssh-client
```

## 2. Connect with a password

```bash
ssh <user>@<server IP>
```

Example:

```bash
ssh labuser@192.168.1.50
```

The first time, answer `yes` to the host fingerprint question. Then enter the password of the account **on the server** (it is not shown while typing). Type `exit` to close the session.

## 3. Create an SSH key (login without password)

```bash
ssh-keygen -t ed25519
```

Press **Enter** to accept the default path (`~/.ssh/id_ed25519`). You can set a passphrase or leave it empty.

This creates two files:

| File | What it is |
|---|---|
| `id_ed25519` | **Private** key. Never share or copy it anywhere. |
| `id_ed25519.pub` | **Public** key. This is the one you copy to servers. |

## 4. Copy the public key to the server

### Server is Ubuntu

```bash
ssh-copy-id <user>@<server IP>
```

Enter the server password one last time.

### Server is Windows

`ssh-copy-id` does not work with Windows servers. Do this instead:

1. Copy the key file to the server:
   ```bash
   scp ~/.ssh/id_ed25519.pub <user>@<server IP>:C:/Users/Public/client_key.pub
   ```
2. On the **server**, open **PowerShell as Administrator** and run one of these:

   * The server account **is an administrator** (the usual case). Keys must go to a shared file:
     ```powershell
     Add-Content C:\ProgramData\ssh\administrators_authorized_keys (Get-Content C:\Users\Public\client_key.pub)
     icacls C:\ProgramData\ssh\administrators_authorized_keys /inheritance:r /grant "*S-1-5-32-544:F" /grant "*S-1-5-18:F"
     Remove-Item C:\Users\Public\client_key.pub
     ```
     `*S-1-5-32-544` is the Administrators group and `*S-1-5-18` is SYSTEM. Using SIDs makes it work on Russian Windows too (where the group is called `Администраторы`).

   * The server account **is not an administrator**:
     ```powershell
     $u = "<user>"
     New-Item -ItemType Directory -Force C:\Users\$u\.ssh | Out-Null
     Add-Content C:\Users\$u\.ssh\authorized_keys (Get-Content C:\Users\Public\client_key.pub)
     Remove-Item C:\Users\Public\client_key.pub
     ```

Test it. It should log in **without** asking for the password:

```bash
ssh <user>@<server IP>
```

## 5. The SSH config file

The client config file stores the connection settings of every server, so you do not type the user, IP and key each time. It is a plain text file without extension. Open (or create) it:

```bash
nano ~/.ssh/config
```

Save with **Ctrl + O**, **Enter**, and exit with **Ctrl + X**. Then set safe permissions (ssh refuses a config that other users can write):

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config
```

Each `Host` block describes one server:

| Field | Meaning |
|---|---|
| `Host` | Any short name you choose. You type it instead of `user@IP`. |
| `HostName` | Real address of the server: IP (LAN or Tailscale `100.64.x.x`) or DNS name. |
| `User` | Account name **on the server**. |
| `Port` | SSH port. `22` unless the server uses another one. |
| `IdentityFile` | Path to your **private** key (from step 3). Omit it to log in with a password. |

Example with several servers:

```
Host lab-pc
    HostName 192.168.1.50
    User labuser
    Port 22
    IdentityFile ~/.ssh/id_ed25519

Host lab-ubuntu
    HostName 100.64.0.12
    User administrator
    IdentityFile ~/.ssh/id_ed25519

Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
```

Rules:

* Indent the lines under `Host` with spaces. Blocks are separated by the next `Host` line.
* `~` means your home folder (`/home/<you>`).
* Lines starting with `#` are comments.

Now connect with just the name:

```bash
ssh lab-pc
```

The same name also works for file copy: `scp file.txt lab-pc:~/`.

## 6. Connect with VS Code (Remote - SSH)

VS Code can open folders and terminals on the server as if they were local. It uses the same `config` file from step 5.

### Install the extension

1. Open VS Code.
2. Press **Ctrl + Shift + X** (Extensions).
3. Search for **Remote - SSH** (publisher: Microsoft, id `ms-vscode-remote.remote-ssh`) and click **Install**.

### Add the host to the config file from VS Code

1. Press **Ctrl + Shift + P** and type `remote`.
2. Choose **Remote-SSH: Open SSH Configuration File...**.
3. Pick your user config file: `/home/<you>/.ssh/config`.
4. Add a `Host` block (like in step 5) and save with **Ctrl + S**.

Alternative: **Remote-SSH: Add New SSH Host...** and type `ssh <user>@<server IP>`. VS Code then writes the block into the config file for you.

### Connect

1. Press **Ctrl + Shift + P**, type `remote` and choose **Remote-SSH: Connect to Host...**.
   You can also click the **><** button in the bottom-left corner of VS Code, or use the **Remote Explorer** icon in the left sidebar.
2. Pick the host name from the list (e.g. `lab-pc`). It is the `Host` name from the config file.
3. A new window opens. On the first connection:
   * choose the **server** operating system: **Linux** (Ubuntu server) or **Windows** (Windows server);
   * answer **Continue** to the fingerprint question;
   * enter the server password if you did not set up a key (step 4).
4. Wait while VS Code installs its server part on the remote host (only the first time; the host needs internet access for this).
5. The bottom-left corner now shows **SSH: lab-pc**. Use **File → Open Folder...** to open a folder **on the server**, and **Terminal → New Terminal** to get a shell on the server.

To disconnect: click **SSH: lab-pc** in the bottom-left corner and choose **Close Remote Connection**.

Extensions in a remote window run on the server. Install the ones you need there (e.g. Python) from the Extensions panel: they show an **Install in SSH: lab-pc** button.

## Troubleshooting

| Error | Fix |
|---|---|
| `Permission denied, please try again.` | Wrong password or wrong user name on the server. |
| `Permission denied (publickey)` | The key is not in the server's `authorized_keys` (step 4), or password login is disabled on the server. |
| VS Code: `Could not establish connection` | Check that `ssh <host>` works in a terminal first. If you chose the wrong OS, press **Ctrl + Shift + P** → **Preferences: Open User Settings (JSON)** and fix `"remote.SSH.remotePlatform"` for that host (`"linux"` or `"windows"`). |
| `Could not resolve hostname` | Wrong IP or `Host` name. Check the server IP with `ipconfig` (Windows) or `hostname -I` (Ubuntu). |
| `Connection timed out` | Server firewall is blocking port 22, or the server is on another network. See `server/ssh/`. |
| `Bad owner or permissions on ~/.ssh/config` | Run `chmod 700 ~/.ssh && chmod 600 ~/.ssh/config`. |
| `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!` | The server was reinstalled or its keys changed. Run `ssh-keygen -R <server IP>` and connect again. |
| `UNPROTECTED PRIVATE KEY FILE!` | Run `chmod 600 ~/.ssh/id_ed25519`. |

Use `ssh -v <user>@<server IP>` to see detailed connection logs.
