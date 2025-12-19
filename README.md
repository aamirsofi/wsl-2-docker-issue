# WSL2 + Docker Disk Bloat (ext4.vhdx) – Complete Recovery & Prevention Guide

This guide documents **the full, real-world recovery process** for WSL2 disk bloat issues, especially when Docker is involved, and **what to do if a step fails**.

It is written to work even when **C: drive is almost full**.

---

## 1. Problem Summary

### Symptoms
- `ext4.vhdx` grows very large (20–100+ GB)
- Disk space is not freed after:
  - deleting Docker images
  - uninstalling Docker Desktop
  - cleaning Linux files
- C: drive shows **0–5 GB free**

### Typical Path
```
C:\Users\<user>\AppData\Local\Packages\CanonicalGroupLimited...\ext4.vhdx
```

### Root Cause
- WSL2 uses a **virtual disk (VHDX)**
- The disk **only grows, never auto-shrinks**
- Windows cannot reclaim freed Linux space
- Docker accelerates this growth

---

## 2. First Diagnostics (Always Start Here)

### Check running WSL distros (PowerShell)
```powershell
wsl -l -v
```

If **any distro is Running**, disk compaction will NOT work.

---

## 3. Try Safe Shrink (May Work, Often Fails)

> Try this only if you still have decent free space.

### 3.1 Zero free space inside Linux
```bash
sudo dd if=/dev/zero of=/zerofile bs=1M status=progress || true
sudo sync
sudo rm -f /zerofile
sudo sync
```

### 3.2 Shut down WSL
```powershell
wsl --shutdown
```

### 3.3 Compact VHDX (PowerShell as Admin)
```powershell
diskpart
```
```text
select vdisk file="<path-to-ext4.vhdx>"
attach vdisk readonly
compact vdisk
detach vdisk
exit
```

### If this works
- ext4.vhdx size reduces → **DONE**

### If this fails (very common)
➡️ Continue to **Section 4**

---

## 4. Free Space Immediately (No Extra Space Needed)

### 4.1 Remove Docker WSL distros (biggest win)
```powershell
wsl --unregister docker-desktop
wsl --unregister docker-desktop-data
```

✔ Frees **10–30 GB instantly**

### 4.2 Remove unused Ubuntu distro (if present)
```powershell
wsl --unregister Ubuntu
```

---

## 5. When Export Fails Due to Low Disk Space

### Typical error
```
There is not enough space on the disk (0x80070070)
```

This happens because WSL tries to export the **bloated filesystem**, not real data.

➡️ Use **home-only backup strategy** (next section).

---

## 6. GUARANTEED FIX (Works Even With 0 Free Space)

### 6.1 Check real data size (inside Ubuntu)
```bash
du -sh /home
df -h /
```

Typical output:
- `/home` = 1–5 GB (real data)
- `/` shows ~1 TB (virtual, ignore)

---

### 6.2 Backup ONLY /home (small & safe)
```bash
cd /home
sudo tar -czf /mnt/c/Users/<user>/home-backup.tar.gz .
```

✔ Usually 1–3 GB
✔ Works even when disk is almost full

Verify:
```bash
ls -lh /mnt/c/Users/<user>/home-backup.tar.gz
```

---

### 6.3 Delete Ubuntu (this frees space instantly)
```powershell
wsl --unregister Ubuntu-22.04
```

💥 ext4.vhdx deleted
💥 20–40 GB reclaimed immediately

---

### 6.4 Reinstall Ubuntu (clean disk)
```powershell
wsl --install -d Ubuntu-22.04
```

---

### 6.5 Restore home data
```bash
cd /home
sudo tar -xzf /mnt/c/Users/<user>/home-backup.tar.gz
sudo chown -R <user>:<user> /home/<user>
```

All configs restored:
- SSH keys
- Git config
- AWS/Azure creds
- Node, npm, Docker configs

---

## 7. Final State Verification

- ext4.vhdx size: **~3–5 GB**
- C: drive free space restored
- Ubuntu works normally

---

## 8. Permanent Prevention (CRITICAL)

Create this file on Windows:

**`C:\Users\<user>\.wslconfig`**
```ini
[wsl2]
memory=4GB
swap=2GB
```

Apply:
```powershell
wsl --shutdown
```

This prevents uncontrolled growth.

---

## 9. Docker-Specific Best Practices

### If Docker disk grows again
```powershell
wsl --unregister docker-desktop
```

Docker Desktop will recreate it cleanly.

### Optional
- Run Docker **inside WSL** without Docker Desktop
- Move WSL to another drive (D:/E:)

---

## 10. Decision Tree (Quick Reference)

- Disk shrinking works? → Stop
- Shrink fails? → Remove Docker WSL
- No space left? → Backup /home only
- Export fails? → DO NOT retry
- Always delete + recreate

---

## 11. Key Truths (Remember These)

- ext4.vhdx **never shrinks automatically**
- Deleting files inside Linux ≠ freeing Windows space
- Docker accelerates WSL disk bloat
- Delete + recreate is the **only guaranteed fix**

---

## 12. Recommended Maintenance

- Monthly: check ext4.vhdx size
- Before heavy Docker use: verify free space
- After big Docker cleanup: recreate Docker WSL if needed

---

**Status:** ✅ Fully recovered and stable

