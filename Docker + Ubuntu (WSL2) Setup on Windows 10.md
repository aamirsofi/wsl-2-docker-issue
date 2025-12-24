# Docker + Ubuntu (WSL2) Setup on Windows 10
## Performance‑Optimized Local Development (Entrata)

---

## 1. Background & Problem Statement

When Docker containers mount source code from the **Windows file system (C:\)**, file I/O becomes extremely slow due to cross‑filesystem communication between Windows and Linux.

This leads to:
- Slow container startup
- Slow hot reloads
- High CPU usage
- Docker disk bloat issues

---

## 2. Why WSL2 File System?

By storing **both Docker containers and source code inside the WSL (Linux) file system**, Docker no longer needs to cross file‑system boundaries.

### Benefits
- 🚀 3–8× faster file access
- 🐳 Stable Docker behavior
- 💾 Fewer disk/inode issues
- ⚡ Faster rebuilds & hot reloads

📖 Reference (Microsoft):
https://learn.microsoft.com/en-us/windows/wsl/filesystems  
See **“File storage and performance across file systems”**

---

## 3. High‑Level Architecture

Windows 10  
↳ WSL2 (Ubuntu 22.04)  
↳ Linux file system (`/var/www`)  
↳ Docker containers using same FS  

---

## 4. Prerequisites

- Windows 10 (Build ≥ 19041)
- Virtualization enabled in BIOS
- Admin access
- Docker Desktop installed

---

## 5. Basic Setup (Step‑by‑Step)

### Step 1: Set WSL2 as Default
Run **PowerShell as Administrator**:
```powershell
wsl --set-default-version 2
```

---

### Step 2: Install Ubuntu 22.04 LTS
Install from Microsoft Store:
https://apps.microsoft.com/store/detail/ubuntu-22041-lts/9PN20MSR04DW

> Tip: Disable VPN if Store fails to open.

---

### Step 3: Enable Ubuntu in Docker Desktop
Docker Desktop → **Settings → Resources → WSL Integration**  
✅ Enable **Ubuntu‑22.04**

---

### Step 4: Where to Clone Repositories (IMPORTANT)

✅ Correct location (inside WSL):
```bash
/var/www
```

```bash
cd /var/www
git clone git@github.com:entrata/docker-env.git ubuntu-env
```

❌ Do NOT clone in:
```
/mnt/c/Users/...
C:\Users\...
```

Reason:
- Poor performance
- Docker volume slowness
- Frequent disk issues

---

## 6. SSH Setup (GitHub)

### 6.1 Which Email to Use?
Use your **personal GitHub account email**.
Organization access comes from **permissions**, not email.

---

### 6.2 Generate SSH Key (inside Ubuntu)
```bash
ssh-keygen -t ed25519 -C "your_personal_email@example.com"
```

---

### 6.3 Add SSH Key to GitHub
```bash
cat ~/.ssh/id_ed25519.pub
```
GitHub → Settings → SSH & GPG Keys → New SSH Key

---

### 6.4 Test SSH
```bash
ssh -T git@github.com
```

Expected:
```
Hi <username>! You've successfully authenticated...
```

---

## 7. Common Git & SSH Errors (Fix‑First Guide)

### ❌ Error:
Password authentication is not supported

✅ Fix:
Use SSH instead of HTTPS.

---

### ❌ Error:
```
fatal: cannot run C:/Windows/System32/OpenSSH/ssh.exe
fatal: unable to fork
```

Cause:
Windows SSH is forced inside WSL.

Fix:
```bash
git config --global --unset-all core.sshCommand
unset GIT_SSH
unset GIT_SSH_COMMAND
```

Verify:
```bash
which ssh
# /usr/bin/ssh
```

---

### ❌ SSH works on Windows but not in WSL

Fix:
```bash
sudo apt update
sudo apt install -y openssh-client
```

---

## 8. Git Configuration for Linux Permissions

Prevent permission‑only diffs:
```bash
git config --global core.fileMode false
git config --global --add safe.directory '*'
```

---

## 9. Prepare Project Directory

```bash
sudo mkdir -p /var/www
sudo chmod -R 0777 /var/www
```

Follow Entrata setup:
https://github.com/entrata/docker-env

---

## 10. Docker Ports (Optional)

Use different ports if running Windows + WSL env together.

Example:
```yaml
ports:
  - 7080:80
```

---

## 11. Start Docker Environment

```bash
docker compose up -d
```

---

## 12. Fix Write Permissions Inside Container

```bash
docker exec -it ubuntu-env-core-1 bash
chmod 777 /var/www/PsCoreConfig
chmod -R 777 /var/www/Logs
chmod -R 777 /tmp
```

---

## 13. Verify Setup

Open browser:
```
http://clientadmin.entrata.localhost:7080/
```

---

## 14. Optional Enhancements

### Show Git Branch in Terminal
Add to `~/.bashrc`:
```bash
parse_git_branch() {
  git branch 2>/dev/null | sed -n '/\* /s///p'
}
export PS1="\u@\h \w (\$(parse_git_branch)) $ "
```

---

## 15. WSL Password Recovery

Reset password:
```bash
sudo passwd <username>
```

Forgot password:
```powershell
wsl -u root
passwd <username>
wsl --shutdown
```

---

## 16. Best Practices Summary

✔ Clone repos inside WSL  
✔ Use Linux Git + Linux SSH  
✔ Avoid `/mnt/c` for Docker projects  
✔ Separate Windows & WSL Git configs  

---

## 17. Outcome

- Faster Docker performance
- Stable local environment
- No repeated cleanup cycles
- Safe & reproducible setup
