
# Setting Up SSH with Password Auto-Login using `sshpass`

This guide explains how to automatically log into an SSH server without typing your password every time, using `sshpass`, split for macOS, Linux, and Windows.

---

# 📦 1. Installation

## macOS

Install `sshpass` with Homebrew:

```bash
brew install hudochenkov/sshpass/sshpass
```

If Homebrew is not installed:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## Linux (Ubuntu/Debian)

Install `sshpass` directly:

```bash
sudo apt update
sudo apt install sshpass
```

## Windows

Use `sshpass` via WSL (Windows Subsystem for Linux). Install WSL and set up Ubuntu, then follow Linux instructions.

Alternatively, use **Putty** and **plink** for scripted SSH (recommended for Windows).

---

# 🔐 2. Store Your Password Securely

Create a hidden password file:

```bash
nano ~/.ssh/pass.txt
```

Inside the file, write **only your SSH password**, then save and exit.

Secure the file:

```bash
chmod 600 ~/.ssh/pass.txt
```

---

# 🚀 3. Create a Smart SSH Function

## macOS/Linux

Edit your shell configuration file:

For `zsh` (default on macOS):

```bash
nano ~/.zshrc
```

For `bash` (Linux or macOS):

```bash
nano ~/.bash_profile
```

Add this **smart function**:

```bash
socksproxy() {
  if [ "$1" = "killall" ]; then
    echo "Killing all autossh processes related to SOCKS proxy on LOCAL_PORT..."
    ps aux | grep autossh | grep LOCAL_PORT | awk '{print $2}' | xargs kill -9
    echo "All matching autossh processes have been killed."
  else
    if pgrep -f "ssh.*-D LOCAL_PORT" > /dev/null; then
      echo "SOCKS proxy is already running."
    else
      echo "Starting SOCKS proxy..."
      sshpass -f ~/.ssh/pass.txt autossh -M 0 -N -D LOCAL_PORT -p REMOTE_PORT USER@SERVER &
      sleep 1
      if pgrep -f "ssh.*-D LOCAL_PORT" > /dev/null; then
        echo "SOCKS proxy started successfully."
      else
        echo "Failed to start SOCKS proxy."
      fi
    fi
  fi
}
```

- **Auto-detect** if proxy is already running.
- **Auto-reconnect** if disconnected (uses `autossh`).
- **Background process** so the terminal is free.

Reload your shell:

```bash
source ~/.zshrc
```
or
```bash
source ~/.bash_profile
```

Use it by simply typing:

```bash
socksproxy
```

## Windows (Putty/Plink Method)

For Windows users without WSL, use a batch file with `plink.exe`:

1. Download [Putty](https://www.putty.org/).
2. Create a batch file `socksproxy.bat`:

```batch
@echo off
plink.exe -ssh USER@SERVER -P REMOTE_PORT -pw yourpassword -D LOCAL_PORT -N
```

3. Double-click to run.

For auto-reconnect, you can use PowerShell scripts or tools like `nssm` to run as a Windows Service.

---

# ⚡ Important Notes

- **Security Warning:** Plain text password files are less secure. Use on trusted machines only.
- Protect your `pass.txt` file with permissions `600`.
- Consider SSH keys for better long-term security.

# 🛡️ Pro Tip

Use tools like `pass` (password manager) or `gpg` to encrypt your password file if you want maximum safety.

---
