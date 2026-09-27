<p>
<img src="../../../assets/icons/ssh.svg" width="48" alt="ssh">&nbsp;
<img src="../../../assets/icons/windows.svg" width="48" alt="windows">
</p>

# OpenSSH Server on Windows

[← Back to all guides](../../../README.md)

## 1. Install OpenSSH Server

1. Press **Win + I** to open **Settings**.
2. Go to **System → Optional features** (Система → Дополнительные компоненты).
3. Click **View features / Show all features** (Показать все компоненты).
4. Open **available features** (Показать доступные компоненты).
5. Type `OpenSSH Server` in the search box.
6. Check **OpenSSH Server** and click **Install**. Wait until the installation finishes.

## 2. Start the service and enable autostart

Open **PowerShell as Administrator** and run:

```powershell
Start-Service sshd
Set-Service -Name sshd -StartupType 'Automatic'
Get-Service sshd
```

Expected output:

```
Status   Name               DisplayName
------   ----               -----------
Running  sshd               OpenSSH SSH Server
```

## 3. Check the firewall rule

```powershell
Get-NetFirewallRule -Name *ssh*
```

Expected output (important fields):

```
Name                          : OpenSSH-Server-In-TCP
DisplayName                   : OpenSSH SSH Server (sshd)
Description                   : Inbound rule for OpenSSH SSH Server (sshd)
DisplayGroup                  : OpenSSH Server
Group                         : OpenSSH Server
Enabled                       : True
Profile                       : Private
Direction                     : Inbound
Action                        : Allow
PrimaryStatus                 : OK
```

## 4. Test the connection locally

```powershell
ssh localhost
```

Enter your Windows account password (switch the keyboard layout to **ENG** first, the password is not shown while typing). On success you get a shell on the PC:

```
user@localhost's password:
Microsoft Windows [Version 10.0.xxxxx.xxxx]
(c) Корпорация Майкрософт (Microsoft Corporation). Все права защищены.

user@LAB-PC C:\Users\user>
```

To connect from another computer: `ssh <user>@<IP of this PC>` (find the IP with `ipconfig`).
