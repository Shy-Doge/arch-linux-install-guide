# Install Index

### 📝 TL;DR / Who this is for

This guide is for anyone who wants a **fully bootable, snapshot-capable Arch Linux system** with Secure Boot enabled.
It’s aimed at users who:

* Prefer using **`archinstall`** for faster setup rather than manual partitioning, especially if facing **time sync / NTP issues** during install.
* Want a **Btrfs root with subvolumes** for easy snapshots and rollbacks.
* Need guidance on **Secure Boot key management** using `sbctl`.
* Appreciate clear, step-by-step instructions and practical examples rather than theoretical documentation.

> ⚠️ If you are following this guide, read everything before starting, verify compatibility with your system, and consult the [ArchWiki](https://wiki.archlinux.org/title/Main_page) for tool-specific details. `sbctl` carries the highest risk if misused.

---

### ⚠️ Disclaimer

> **⚠️ Warning:** This is a guide for myself so I don't forget the steps I did on my own PC.
> If you follow it, ***please*** read everything first and make sure your system matches the assumptions in this guide. The only tool I give a serious disclaimer for is `sbctl` because improper use can brick your system.

---

### 🗂️ Document Index

This guide is broken into stages that reflect the actual install flow I used. Each section builds on the previous one, starting with pre-install fixes and ending in a fully bootable, snapshot-capable Arch Linux system.

> 🔧 **Start with `1. PreSetup`** — if you skip that, time sync or keyring issues may cause `archinstall` or package installs to fail.

1. **[1. PreSetup](1-PreSetup.md)**

    *Get your system ready for a smooth Arch installation by fixing time sync issues, disabling Secure Boot temporarily, and initializing Pacman's keyring.*
    
    - **Edit Timesync Configuration**
    *(Prevents time-related install failures due to blocked NTP servers)*
        
    - **Enable and Sync Time**
    *(Ensures system clock is accurate — required for HTTPS and secure installs)*
        
    - **Update Pacman Keys**
    *(Avoids signature verification errors during package installs)*
	    
2. **[2. Install with Btrfs](2-Install-with-Btrfs.md)**
    
    - **Manual partitioning and subvolume creation**
        
    - **GRUB setup and Secure Boot using `sbctl`**
        
    - **Includes `/swap` subvolume and swapfile at `/swap/swapfile`**
        
3. **[3. SecureBoot on Arch using SBCTL](3-SecureBoot-SBCTL.md)**
    
    - **Signing kernel and EFI binaries**
        
    - **Key creation and firmware enrollment**
        
    - **Troubleshooting quirks**
        
4. **[4. Btrfs Snapshots and Rollbacks](4-Btrfs-Snapshots.md)**
    
    - **Automatic and manual snapshot creation**
        
    - **GRUB boot entry integration optional**
        
    - **Pacman integration with `snap-pac`**
        

---

### 💾 Partition Layout (example)

| Device    | Size  | Type  | Mount Point | Description          |
| --------- | ----- | ----- | ----------- | -------------------- |
| /dev/sda1 | 500MB | FAT32 | `/boot/efi` | EFI partition (GRUB) |
| /dev/sda2 | 1GB   | ext4  | `/boot`     | Kernel + GRUB files  |
| /dev/sda3 | ~2TB  | Btrfs | `/`         | Main Btrfs volume    |

---

### 📁 Btrfs Subvolumes

|Subvolume|Mount Point|Purpose|
|---|---|---|
|`@`|`/`|Root filesystem|
|`@home`|`/home`|User data|
|`@log`|`/var/log`|Logs|
|`@pkg`|`/var/cache/pacman/pkg`|Pacman cache|
|`@snapshots`|`/.snapshots`|Snapper-managed snapshots|
|`@swap`|`/swap`|Swapfile stored here|

---

### 🧊 Swap File Location

- Subvolume: `@swap`
    
- Mount point: `/swap`
    
- Swap file path: `/swap/swapfile`
    
- Created using `fallocate` + `chattr +C` to disable COW
    
