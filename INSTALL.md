# Installation Guide

Everything you need to install a boot animation from BootVault on any Android device.

---

## Before You Start

**Find your installation path.** Android searches for `bootanimation.zip` in this order — the first one found wins:

```
1. /data/local/bootanimation.zip       ← no root needed, lost on factory reset
2. /product/media/bootanimation.zip    ← Pixel 4a+, some AOSP ROMs
3. /oem/media/bootanimation.zip        ← rare, some OEM builds
4. /system/media/bootanimation.zip     ← standard, survives factory reset
```

**Find your device's path:**
```bash
adb shell find /system /product /oem -name "bootanimation.zip" 2>/dev/null
```

> **Samsung OneUI note:** Stock OneUI uses a proprietary `.qmg` format — standard `bootanimation.zip` only works if you're on a custom ROM (LineageOS, crDroid, etc.). If you're on stock OneUI, you need a Magisk module that patches the bootanim service.

---

## Method 1 — No Root (ADB to /data/local)

Works on any device with USB Debugging enabled. Does **not** survive a factory reset.

**Requirements:** ADB installed on your computer, USB Debugging enabled on device.

```bash
# 1. Push the file
adb push bootanimation.zip /data/local/bootanimation.zip

# 2. Set permissions
adb shell chmod 644 /data/local/bootanimation.zip

# 3. Reboot
adb reboot
```

**To revert:**
```bash
adb shell rm /data/local/bootanimation.zip
adb reboot
```

> This works because `/data/local` takes priority over `/system/media`. No root, no remounting, nothing — just push and reboot.

---

## Method 2 — ADB + Root (/system/media, survives factory reset)

**Requirements:** Rooted device, ADB.

```bash
# 1. Remount system as read-write
adb shell su -c "mount -o rw,remount /system"

# 2. Back up original
adb shell su -c "cp /system/media/bootanimation.zip /sdcard/bootanimation.zip.bak"

# 3. Push and move
adb push bootanimation.zip /sdcard/bootanimation.zip
adb shell su -c "cp /sdcard/bootanimation.zip /system/media/bootanimation.zip"

# 4. Set permissions
adb shell su -c "chmod 644 /system/media/bootanimation.zip"
adb shell su -c "chown root:root /system/media/bootanimation.zip"

# 5. Reboot
adb reboot
```

If your device uses `/product/media/` instead of `/system/media/`:
```bash
adb shell su -c "mount -o rw,remount /product"
adb shell su -c "cp /sdcard/bootanimation.zip /product/media/bootanimation.zip"
adb shell su -c "chmod 644 /product/media/bootanimation.zip"
```

---

## Method 3 — Recovery File Manager (TWRP / OrangeFox)

No need to boot into the OS. Good for fixing bootloops caused by a bad animation.

**Steps:**

1. Boot into recovery (`adb reboot recovery` or Power + Vol Down)
2. In recovery: **Mount → System** (and Mount → Product if needed)
3. Push the file via ADB:

```bash
adb push bootanimation.zip /system/media/bootanimation.zip
adb shell chmod 644 /system/media/bootanimation.zip
adb shell chown root:root /system/media/bootanimation.zip
```

Or use the recovery's built-in file manager:
- TWRP: **Advanced → File Manager**
- OrangeFox: **File Manager** in the main menu

Navigate to `/system/media/`, delete the old `bootanimation.zip`, paste the new one, set permissions to `644`.

4. Reboot to system.

---

## Method 4 — Root File Explorer App (no PC needed)

**Requirements:** Rooted device, root-capable file manager (MT Manager, Solid Explorer, MiXplorer, Root Explorer).

1. Copy `bootanimation.zip` to your phone's internal storage
2. Open the file manager, enable root access
3. Navigate to `/system/media/` (or `/product/media/`)
4. Long-press the existing `bootanimation.zip` → rename it to `bootanimation.zip.bak`
5. Copy your new file here
6. Long-press it → Properties → Permissions → set to `rw-r--r--` (644)
7. Reboot

---

## Permissions Reference

Always verify after installing:

```bash
adb shell ls -la /system/media/bootanimation.zip
# Expected output:
# -rw-r--r-- 1 root root <size> <date> /system/media/bootanimation.zip
```

| Value | Meaning |
|-------|---------|
| `644` | ✅ Correct — system will read it |
| `600` | ❌ Too restrictive — system ignores it |
| `777` | ⚠️ Works but not recommended |

---

## Troubleshooting

**Animation doesn't play / stock animation shows:**
- Wrong path — run the find command at the top to locate the active file
- Wrong permissions — verify with `ls -la`
- ZIP is compressed — must be packed with STORE (no compression), not DEFLATE

**Black screen on boot:**
- `desc.txt` has wrong resolution or format
- Image filenames are not sequential (must start from `0000` or `00000`)
- Boot into recovery, delete the file, reboot

**`/system` is read-only, remount fails:**
- Device uses dynamic partitions (Android 10+) — remounting system R/W is blocked
- Use the `/data/local` method (Method 1) or recovery method instead

**Animation reverts after reboot (stock ROM):**
- The ROM is restoring its own file — use `/data/local` path instead, it has higher priority

**Verify your ZIP is packed correctly (no compression):**
```bash
unzip -v bootanimation.zip | awk '{print $5, $8}' | head -5
# "Stored" = ✅ correct
# "Deflated" = ❌ will cause black screen or be ignored
```

Repack if needed:
```bash
cd your-animation-folder
zip -0 -r ../bootanimation.zip .
```

---

## Restoring the Original

If you made a backup:
```bash
# From /data/local (no root)
adb shell rm /data/local/bootanimation.zip

# From /system/media (with root)
adb shell su -c "cp /sdcard/bootanimation.zip.bak /system/media/bootanimation.zip"
```

From recovery if the device doesn't boot — navigate to `/system/media/`, delete `bootanimation.zip`, reboot.
