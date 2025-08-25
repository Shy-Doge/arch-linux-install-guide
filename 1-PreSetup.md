### 🕒 Fix NTP Sync Issues (Pre-Install)

If your DNS blocks the default NTP servers, `archinstall` may fail due to time sync errors. Let’s fix that before doing anything.

---

### 🧱 Disabling Secure boot

1. **Disable Secure Boot** in BIOS temporarily.
    
2. **Boot Arch ISO.**

---

#### ✏️ Edit Timesync Configuration

Open the `systemd-timesyncd` config file:

```bash
sudo nano /etc/systemd/timesyncd.conf
```

Uncomment and update the `NTP=` and `FallbackNTP=` lines (or add them if missing).

```
NTP=europe.pool.ntp.org
FallbackNTP=time.google.com <other-ntp> <other-ntp>
```

Replace `<other-ntp>` with additional NTP servers if needed. Keep the originals just in case.

---

#### ⚙️ Enable and Sync Time

Enable and start the time sync service:

```bash
sudo systemctl enable --now systemd-timesyncd
```

Then force time sync:

```bash
timedatectl set-ntp true
```

Check the status:

```bash
timedatectl status
```

> ✅ **Ensure `System clock synchronized` shows `yes`.** This confirms your time sync is now working.

---

### 🔐 Update Pacman Keys

To ensure `archinstall` runs correctly, initialize and populate the pacman keyring.

```bash
sudo pacman-key --init
sudo pacman-key --populate
```

> 🛠️ **This step is required:** This ensures the package manager trusts official Arch package signatures before installation.

Next [2. Install with Btrfs](2-Install-with-Btrfs.md)