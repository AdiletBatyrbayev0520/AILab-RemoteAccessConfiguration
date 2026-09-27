# OpenSSH Client on Windows

The client is the computer you connect **from**. The server must already be set up (see `server/ssh/`).

## 1. Install OpenSSH Client

On Windows 10/11 the client is usually installed already. Check it in PowerShell:

```powershell
ssh -V
```

Expected output (version may differ):

```
OpenSSH_for_Windows_9.5p1, LibreSSL 3.8.2
```

If you get `ssh : The term 'ssh' is not recognized...`, install it:

1. Press **Win + I** to open **Settings**.
2. Go to **System → Optional features** (Система → Дополнительные компоненты).
3. Click **View features / Show all features** (Показать все компоненты).
4. Open **available features** (Показать доступные компоненты).
5. Type `OpenSSH Client` in the search box.
6. Check **OpenSSH Client** and click **Install**.
7. Close and reopen PowerShell, then run `ssh -V` again.

## 2. Connect with a password

```powershell
ssh <user>@<server IP>
```

Example:

```powershell
ssh labuser@192.168.1.50
```

The first time, answer `yes` to the host fingerprint question. Then enter the password of the account **on the server** (switch the keyboard layout to **ENG** first, the password is not shown while typing). Type `exit` to close the session.

## 3. Create an SSH key (login without password)

```powershell
ssh-keygen -t ed25519
```

Press **Enter** to accept the default path (`C:\Users\<you>\.ssh\id_ed25519`). You can set a passphrase or leave it empty.

This creates two files:

| File | What it is |
|---|---|
| `id_ed25519` | **Private** key. Never share or copy it anywhere. |
| `id_ed25519.pub` | **Public** key. This is the one you copy to servers. |

## 4. Copy the public key to the server

Windows has no `ssh-copy-id`, so use one of the commands below.

### Server is Ubuntu

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh <user>@<server IP> "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

### Server is Windows

1. Copy the key file to the server:
   ```powershell
   scp $env:USERPROFILE\.ssh\id_ed25519.pub <user>@<server IP>:C:/Users/Public/client_key.pub
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

```powershell
ssh <user>@<server IP>
```

## 5. Create a shortcut in the SSH config file

Instead of typing the user and IP every time, open the client config file:

```powershell
notepad $env:USERPROFILE\.ssh\config
```

Add one block per server and save. Make sure the file has **no `.txt` extension**:

```
Host lab-pc
    HostName 192.168.1.50
    User labuser
    Port 22
    IdentityFile ~/.ssh/id_ed25519
```

Now connect with just:

```powershell
ssh lab-pc
```

The same name also works for file copy: `scp file.txt lab-pc:C:/Users/Public/`.

## Troubleshooting

| Error | Fix |
|---|---|
| `Permission denied, please try again.` | Wrong password, or the RU keyboard layout was active while typing. Switch to **ENG**. |
| `Could not resolve hostname` | Wrong IP or `Host` name. Check the server IP with `ipconfig` (Windows) or `hostname -I` (Ubuntu). |
| `Connection timed out` | Server firewall is blocking port 22, or the server is on another network. See `server/ssh/`. |
| `Bad owner or permissions on ...\.ssh\config` | The `.ssh` folder has wrong permissions. Run the commands below in PowerShell as Administrator. |
| `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!` | The server was reinstalled or its keys changed. Run `ssh-keygen -R <server IP>` and connect again. |

Fix for `Bad owner or permissions`:

```powershell
$sid = [System.Security.Principal.WindowsIdentity]::GetCurrent().User.Value
$d   = "$env:USERPROFILE\.ssh"
icacls $d /setowner "*$sid" /t /c
icacls $d /grant:r "*${sid}:(OI)(CI)F" "*S-1-5-18:(OI)(CI)F" "*S-1-5-32-544:(OI)(CI)F"
icacls $d /inheritance:r
icacls "$d\*" /reset /t /c
```
