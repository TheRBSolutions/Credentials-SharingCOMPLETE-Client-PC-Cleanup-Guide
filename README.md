# Credentials-SharingCOMPLETE-Client-PC-Cleanup-Guide

# **COMPLETE Client PC Cleanup Guide**

Run these commands in order on the **Client PC (DESKTOP-U852KNO)**:

---

## **Step 1: Clear ALL Credential Layers**

```powershell
# 1. Clear saved credentials (GUI)
rundll32.exe keymgr.dll,KRShowKeyMgr
# Manually remove all MUSTAFA/172.99.0.55 entries, then close

# 2. Clear credentials (CLI - all methods)
cmdkey /list
cmdkey /delete:MUSTAFA
cmdkey /delete:172.99.0.55
cmdkey /delete:TERMSRV/MUSTAFA

# 3. Clear ACTIVE SMB sessions (critical!)
net use
net use * /delete /y

# 4. Clear Kerberos tickets (even on workgroup)
klist
klist purge

# 5. Flush DNS
ipconfig /flushdns
```

---

## **Step 2: Disable Guest Fallback (Prevents "Contact Admin" Error)**

```cmd
reg add HKLM\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters /v AllowInsecureGuestAuth /t REG_DWORD /d 0 /f
```

---

## **Step 3: Restart Network Services**

```powershell
net stop workstation
net start workstation
```

---

## **Step 4: Verify Clean State**

```powershell
# Should return nothing:
net use
cmdkey /list | findstr /i "MUSTAFA 172.99"
klist
```

---

## **Step 5: Reconnect with Fresh Credentials**

**Method A - File Explorer:**
```
\\172.99.0.55\HassanShare
```

**Method B - Force Password Prompt:**
```cmd
net use Z: \\172.99.0.55\HassanShare /user:MUSTAFA\user_read *
```
The `*` forces a password prompt.

---

## **Step 6: Verify Connection (On Main PC - MUSTAFA)**

```powershell
# Check who's actually connected
Get-SmbSession | Select ClientComputerName, ClientUserName
```

Should show:
- ClientComputerName: `DESKTOP-U852KNO`
- ClientUserName: `MUSTAFA\user_read` (or `user_full`)

---

## **Quick Reset Script (Save as .bat for future)**

```batch
@echo off
echo === Cleaning Network Credentials ===
cmdkey /delete:MUSTAFA 2>nul
cmdkey /delete:172.99.0.55 2>nul
net use * /delete /y
klist purge 2>nul
ipconfig /flushdns
echo === Done! Now reconnect to share ===
pause
```

---

## **Why 3 Layers Matter**

| Layer | Command | What It Clears |
|-------|---------|----------------|
| **Saved Credentials** | `cmdkey /delete` | Stored passwords |
| **Active Sessions** | `net use /delete` | Open SMB connections |
| **Kerberos Tickets** | `klist purge` | Cached auth tokens |

**Missing even ONE layer = "Contact Admin" errors persist.**

---

## **After All This:**

1. Restart Client PC (recommended)
2. Connect to `\\172.99.0.55\HassanShare`
3. Enter: `MUSTAFA\user_read` + password
4. ✅ Check "Remember credentials"

**This is the complete, production-grade cleanup.**
