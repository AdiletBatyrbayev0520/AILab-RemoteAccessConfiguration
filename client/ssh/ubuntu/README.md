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

## 5. Create a shortcut in the SSH config file

Instead of typing the user and IP every time, edit the client config file:

```bash
nano ~/.ssh/config
```

Add one block per server, then save with **Ctrl + O**, **Enter**, and exit with **Ctrl + X**:

```
Host lab-pc
    HostName 192.168.1.50
    User labuser
    Port 22
    IdentityFile ~/.ssh/id_ed25519
```

Set safe permissions (ssh refuses a config that other users can write):

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config
```

Now connect with just:

```bash
ssh lab-pc
```

The same name also works for file copy: `scp file.txt lab-pc:~/`.

## Troubleshooting

| Error | Fix |
|---|---|
| `Permission denied, please try again.` | Wrong password or wrong user name on the server. |
| `Permission denied (publickey)` | The key is not in the server's `authorized_keys` (step 4), or password login is disabled on the server. |
| `Could not resolve hostname` | Wrong IP or `Host` name. Check the server IP with `ipconfig` (Windows) or `hostname -I` (Ubuntu). |
| `Connection timed out` | Server firewall is blocking port 22, or the server is on another network. See `server/ssh/`. |
| `Bad owner or permissions on ~/.ssh/config` | Run `chmod 700 ~/.ssh && chmod 600 ~/.ssh/config`. |
| `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!` | The server was reinstalled or its keys changed. Run `ssh-keygen -R <server IP>` and connect again. |
| `UNPROTECTED PRIVATE KEY FILE!` | Run `chmod 600 ~/.ssh/id_ed25519`. |

Use `ssh -v <user>@<server IP>` to see detailed connection logs.
