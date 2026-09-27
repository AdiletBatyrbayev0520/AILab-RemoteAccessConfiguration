# OpenSSH Server on Ubuntu

## 1. Install OpenSSH Server

Open a terminal (**Ctrl + Alt + T**) and run:

```bash
sudo apt update
sudo apt install -y openssh-server
```

## 2. Start the service and enable autostart

```bash
sudo systemctl enable --now ssh
sudo systemctl status ssh
```

Expected output (important line):

```
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; enabled; preset: enabled)
     Active: active (running)
```

Press **q** to leave the status view.

> On Ubuntu 22.10 and newer, SSH may be started by `ssh.socket` (socket activation). Then `ssh.service` can show `inactive (dead)` until the first connection. That is normal. Check with `sudo systemctl status ssh.socket`.

## 3. Open the firewall (if UFW is enabled)

```bash
sudo ufw allow OpenSSH
sudo ufw status
```

Expected output (if UFW is active):

```
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
```

If it says `Status: inactive`, the firewall is off and nothing blocks SSH.

## 4. Test the connection locally

```bash
ssh localhost
```

The first time, answer `yes` to the host fingerprint question, then enter your Ubuntu user password (it is not shown while typing). On success you get a shell:

```
user@localhost's password:
Welcome to Ubuntu ...

user@hostname:~$
```

Type `exit` to close the session.

## 5. Connect from another computer

Find the IP address of the Ubuntu PC:

```bash
hostname -I
```

Then, from another computer (Windows PowerShell, macOS or Linux terminal):

```bash
ssh <user>@<IP of this PC>
```

## Useful commands

| Action | Command |
|---|---|
| Restart after config changes | `sudo systemctl restart ssh` |
| Server config file | `/etc/ssh/sshd_config` |
| View login logs | `sudo journalctl -u ssh -n 50` |
| Check that port 22 is listening | `sudo ss -tlnp \| grep :22` |
