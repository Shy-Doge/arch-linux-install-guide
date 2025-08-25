### 🧩 Partition Setup (pre-install)

You’ll manually partition like this:

| Partition | Size   | Format | Mount Point        | Notes                                                                                                               |
| --------- | ------ | ------ | ------------------ | ------------------------------------------------------------------------------------------------------------------- |
| EFI       | 500 MB | FAT32  | `/boot/efi`        | Use a dedicated EFI partition (separate from the Windows EFI). Mark this partition with the `esp` and `boot` flags. |
| Boot      | 1 GB   | ext4   | `/boot`            | Used for GRUB + kernels                                                                                             |
| Btrfs     | ~2 TB  | btrfs  | `/` (with subvols) | Root and subvolumes go here                                                                                         |
| Swapfile  | 36 GB  | —      | `/swap/swapfile`   | Nested swapfile inside subvolume                                                                                    |

> ✅ **No swap partition — we will use a swap file (not a swap partition) inside a Btrfs subvolume. This gives the advantage of easily resizing the swap and won't mess with the snapshots system later on.**

>**⚠️ Warning:** If your `archinstall` doesn’t allow you to set a partition as `esp`, use the suggested layout instead or create the scheme manually with `fdisk`.
>The suggested layout combines the `EFI` and `Boot` partitions into one, which will be marked as `esp` automatically.
>If you plan to dual boot with Windows or use full-disk encryption, it’s recommended to keep separate `EFI` and `Boot` partitions.

---

### 🧱 Install Arch Linux (no Secure Boot yet)

1. Do the Steps from [1. PreSetup](1-PreSetup.md).
    
2. **Run ****`archinstall`****.
    
    Use **custom partitioning** with the layout above.
    
    Under mount options, enable Btrfs subvolumes:
    
    ```
    @           → /
    @home       → /home
    @log        → /var/log
    @pkg        → /var/cache/pacman/pkg
    @snapshots  → /.snapshots
    @swap       → /swap
    ```
    
    - Choose **GRUB** as the bootloader.
        
    - Set your timezone, locale, hostname, and user.
        
    - Set the networking to use NetworkManager.
        
3. **Complete the install and reboot into Arch.**
    

---

### 💾 Create & Enable Swapfile (inside @swap subvolume)

```bash
sudo mkdir -p /swap
sudo chattr +C /swap
sudo fallocate -l 36G /swap/swapfile
sudo chmod 600 /swap/swapfile
sudo mkswap /swap/swapfile
sudo swapon /swap/swapfile
```

Add to `/etc/fstab`:

> ⚠️ Make sure COW (Copy-on-Write) is disabled on the swap file directory (`/swap`) by using `chattr +C` **before** creating the file.

```
/swap/swapfile none swap defaults 0 0
```

Next [3. SecureBoot on Arch using SBCTL](3-SecureBoot-SBCTL.md)