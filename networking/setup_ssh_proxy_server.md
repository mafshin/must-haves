# SSH Proxy Setup on Ubuntu

This following script sets up a secure, limited SSH user on your Ubuntu server who can only use **SSH port forwarding (e.g., SOCKS proxy or tunnels)** on your desired **SSH_PORT** , without any shell access. 

It also disables shell access for `root` and `proxyuser`.

---

## 🔧 Setup Instructions

1. **Run the script on your Ubuntu server:**

   ```bash
   sudo bash setup_ssh_proxy_server.sh
   ```

   Or simply copy and paste the contents of the script below into your terminal:

   ```bash
   #!/bin/bash

   set -e

   # Variables
   SSH_PORT=443
   PROXY_USER=proxyuser

   echo "[+] Installing OpenSSH Server if not present"
   apt-get update
   apt-get install -y openssh-server

   echo "[+] Creating proxy user"
   id "$PROXY_USER" &>/dev/null || useradd -m -s /usr/sbin/nologin "$PROXY_USER"

   echo "[+] Setting a password for $PROXY_USER (you can change it later)"
   echo "$PROXY_USER:changeme123" | chpasswd

   echo "[+] Locking shell access for root and proxyuser"
   usermod -s /usr/sbin/nologin root || true
   usermod -s /usr/sbin/nologin proxyuser || true

   echo "[+] Configuring SSH daemon"

   # Backup sshd_config
   cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak.$(date +%s)

   # Change default SSH port to 443
   sed -i 's/^#Port.*/Port 443/' /etc/ssh/sshd_config
   sed -i 's/^Port .*/Port 443/' /etc/ssh/sshd_config

   # Disable root login
   sed -i 's/^#*PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config

   # Allow only this user
   echo "AllowUsers $PROXY_USER" >> /etc/ssh/sshd_config

   # Force command to prevent shell use
   echo "Match User $PROXY_USER" >> /etc/ssh/sshd_config
   echo "  ForceCommand echo 'This account is for SSH port forwarding only.'" >> /etc/ssh/sshd_config
   echo "  PermitTunnel no" >> /etc/ssh/sshd_config
   echo "  AllowTcpForwarding yes" >> /etc/ssh/sshd_config
   echo "  X11Forwarding no" >> /etc/ssh/sshd_config

   echo "[+] Restarting SSH service"
   systemctl restart ssh

   echo "[✔] Done. User '$PROXY_USER' is ready for SSH port forwarding on port $SSH_PORT"
   echo "[!] IMPORTANT: Change the password of '$PROXY_USER' with 'passwd $PROXY_USER'"
   ```

2. The script will:

   * Install OpenSSH server (if needed)
   * Create a restricted user: `proxyuser` (default password: `changeme123`)
   * Change SSH port to `443`
   * Disable shell access for `root` and `proxyuser`
   * Configure SSH to allow only port forwarding for `proxyuser`

3. **Reboot the server** for all changes to apply:

   ```bash
   sudo reboot
   ```

4. **(Important)** Change the default password:

   ```bash
   sudo passwd proxyuser
   ```

---

## How to Connect

To connect using SOCKS5 proxy from your device:

```bash
ssh -N -D 1080 -p 443 proxyuser@your.server.ip
```

This opens a local SOCKS5 proxy at `localhost:1080`, which you can use in browsers or apps.

---

## 💻 Client Applications

Use any of the following clients to connect:

### 🪟 Windows

* **Bitvise SSH Client**
  🔗 [https://bitvise.com/ssh-client-download](https://bitvise.com/ssh-client-download)
  A powerful Windows GUI for SSH tunnels and port forwarding.

### 📱 iOS

* **NPV Tunnel**
  🔗 [https://apps.apple.com/us/app/npv-tunnel/id1629465476](https://apps.apple.com/us/app/npv-tunnel/id1629465476)
  Supports SSH tunnels, custom payloads, and proxies.

### 🤖 Android

* **HTTP Injector** by Evozi
  🔗 [https://play.google.com/store/apps/details?id=com.evozi.injector\&hl=en](https://play.google.com/store/apps/details?id=com.evozi.injector&hl=en)
  Popular Android app for SSH tunneling, custom proxies, and payload injection.

  
## ⚠️ Warning

* Sharing this account may lead to **abuse**. If users use the SSH tunnel for illegal activities, your server may be **blacklisted or suspended**.
* If someone uses this tunnel to send excessive traffic or perform a **DDoS or spam relay**, your server may become unusable or be blocked by your provider.
* Always restrict access to trusted parties and monitor usage regularly.

