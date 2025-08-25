### 📦 What is `sbctl`?

`sbctl` is a Secure Boot key and binary manager tailored for Linux users. It helps create, enroll, and manage Secure Boot keys, and sign your EFI binaries — including bootloaders and kernels.

> ⚠️ **Warning:** Replacing the platform keys with your own can end up bricking hardware on some machines, including laptops, making it impossible to get into the firmware settings to rectify the situation. This is due to the fact that some device (e.g GPU) firmware ([OpROMs](https://en.wikipedia.org/wiki/OpROM "wikipedia:OpROM")), that get executed during boot, [are signed using Microsoft 3rd Party UEFI CA certificate](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot#Microsoft_Windows) or vendor certificates. This is the case in many [Lenovo Thinkpad X, P and T series laptops](https://forums.lenovo.com/t5/Other-Linux-Discussions/Reports-of-custom-secure-boot-keys-bricking-recent-X-P-and-T-series-laptops/m-p/5105571) which uses the Lenovo CA certificate to sign UEFI applications and firmware.
> - [ArchWiki](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot)

---

## 🧰 Step-by-step Guide

### 1. Install `sbctl`

```bash
sudo pacman -S sbctl
```

> ⚠️ Make sure you have SecureBoot disabled and delete the keys from the bios!

---

### 2. Initialize Secure Boot Keys

```bash
sudo sbctl create-keys
```

This generates the Platform Key (PK), Key Exchange Key (KEK), and Signature Database (db).

---

### 3. Enroll Keys Into Firmware

```bash
sudo sbctl enroll-keys -m
```

This updates your firmware’s Secure Boot key database with your own keys, while also keeping Microsoft’s keys (required for some hardware like GPUs).

---

### 4. Sign EFI Binaries

To sign the GRUB bootloader:

```bash
sudo sbctl sign /boot/efi/EFI/GRUB/grubx64.efi
```

Sign your kernel:

```bash
sudo sbctl sign /boot/vmlinuz-linux
```

Sign GRUB core and fallback loaders (if used):

```bash
sudo sbctl sign /boot/efi/EFI/BOOT/BOOTX64.EFI
sudo sbctl sign /boot/grub/x86_64-efi/core.efi
sudo sbctl sign /boot/grub/x86_64-efi/grub.efi
```

📌 **Tip:** To sign everything automatically:

```bash
sudo sbctl sign --all
```

---

### 5. Enable Secure Boot in BIOS/UEFI

- Reboot and enter your firmware setup.
    
- Enable Secure Boot.
    
- Save changes and reboot.
    

---

### 6. Verify Everything Works

Check status of your Secure Boot and signed binaries:

```bash
sbctl status
```

---

### 🛠️ Troubleshooting

If you end up in GRUB rescue mode, your firmware may have limited support for user-managed Secure Boot keys.

When running `sbctl status`, if you see:

```
Your firmware has quirks
```

...you may need to follow vendor-specific instructions provided by `sbctl`.

Next [4. Btrfs Snapshots and Rollbacks](4-Btrfs-Snapshots.md)