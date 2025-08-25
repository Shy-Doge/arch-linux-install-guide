### 📦 Install Required Packages

Start by installing Snapper, Snap-pac, and Btrfs Assistant (a GUI for managing snapshots):

```bash
sudo pacman -S snapper snap-pac
yay -S btrfs-assistant
```

---

### 🏗️ Subvolume Layout Assumptions

These instructions assume you created these Btrfs subvolumes during Arch install:

```
@           → /
@snapshots  → /.snapshots
@home       → /home
@log        → /var/log
@pkg        → /var/cache/pacman/pkg
@swap       → /swap
```

> Snapshots will be stored outside of `@`, ensuring rollbacks don't erase your snapshot history.

> ⚠️ If any of these subvolumes are missing, mount your Btrfs volume and create them manually.

---

### ⚙️ Snapper Configuration for Root Subvolume

1. **Unmount and remove existing Snapshots mount point (if mounted):**
    

```bash
sudo umount /.snapshots
sudo rm -rf /.snapshots
```

2. **Create Snapper config for `/` (root):**
    

```bash
sudo snapper -c root create-config /
```

This will:

- Create `/etc/snapper/configs/root`
    
- Add an entry to `/etc/conf.d/snapper`
    
- Generate `.snapshots` subvolume under `@` (we'll remove it next)
    

3. **Remove Snapper's `.snapshots` subvolume to keep using your pre-created `@snapshots`:**
    

```bash
sudo btrfs subvolume delete /.snapshots
```

4. **Re-create mountpoint and re-mount:**
    

```bash
sudo mkdir /.snapshots
sudo mount -a
```

5. **Fix permissions:**
    

```bash
sudo chmod 750 /.snapshots
sudo chown :wheel /.snapshots
```

This ensures only root and members of the `wheel` group can manage snapshots.

6. **Ensure `@snapshots` is mounted via `/etc/fstab`:**

Add this line to `/etc/fstab` (replace UUID with the one for your root Btrfs partition).  
Find your UUID with `blkid` if you’re unsure.

```
UUID=<your-root-uuid> /.snapshots btrfs subvol=@snapshots,defaults,noatime,compress=zstd 0 0
```
  
Then apply it:

```bash
sudo mount -a
```

---

### 📸 Manual Snapshot Example

```bash
sudo snapper -c root create -d "Base system install"
```

---

### ⏱️ Enable Automatic Timeline Snapshots

1. **Edit `/etc/snapper/configs/root`:**
    

Example settings:

```ini
ALLOW_USERS="yourusername"
TIMELINE_MIN_AGE="1800"
TIMELINE_LIMIT_HOURLY="5"
TIMELINE_LIMIT_DAILY="7"
TIMELINE_LIMIT_WEEKLY="0"
TIMELINE_LIMIT_MONTHLY="0"
TIMELINE_LIMIT_YEARLY="0"
```

2. **Enable timers:**
    

```bash
sudo systemctl enable --now snapper-timeline.timer
sudo systemctl enable --now snapper-cleanup.timer
```

---

### 📦 Pacman Integration

Thanks to `snap-pac`, snapshots are automatically created before and after `pacman` installs/removals.

Example:

```bash
sudo pacman -S tree
```

Check:

```bash
sudo snapper -c root list
```

---

### 🔍 View Existing Snapshots and Subvolumes

```bash
sudo btrfs subvolume list /
```

You’ll see entries like:

```
ID 258 top level 5 path @snapshots
ID 266 top level 258 path @snapshots/1/snapshot
ID 267 top level 258 path @snapshots/2/snapshot
```

---

### ✅ Configure GRUB Snapshot Integration

While it's technically possible to enable snapshot entries in the GRUB menu, I’ve chosen not to do so to avoid cluttering GRUB—especially since I'm using Secure Boot, which can complicate things further.

---

### ↩️ Restore Snapshots

If you ever need to restore a snapshot, you can do so using a live USB. Simply boot into the live environment, install `snapper`, and then mount your filesystem. From there, you can use `snapper` (or manually with `btrfs subvolume delete`/`snapshot`) to roll back to the desired state.