# Bare-metal Arch → this desktop

The full path from a blank disk to this Hyprland setup. The OS install itself is
**not scripted** (partitioning someone's disk from a script is how you lose data) —
use `archinstall`, then this repo handles the rest.

## 1. Base Arch (manual, once)

Boot the Arch ISO and run:

```bash
archinstall
```

Reasonable choices for this setup:
- **Profile:** Minimal (we install the desktop ourselves) — or Desktop → none.
- **Audio:** Pipewire.
- **Network:** **NetworkManager** (important — this repo assumes NM, not networkd).
- **Bootloader:** systemd-boot or GRUB (either is fine).
- Create your user, enable sudo.

Reboot into the new system, log in on a TTY, connect (`nmtui`), then:

```bash
sudo pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay-bin.git && cd yay-bin && makepkg -si  # AUR helper
```

## 2. This repo

```bash
git clone --recurse-submodules https://github.com/sitish-kumar/dotfiles.git ~/Projects/dotfiles
cd ~/Projects/dotfiles
./install.sh          # packages + configs + plugins/fork  (see README.md)
./system/system-setup.sh   # system tweaks (network/keyring/etc) — idempotent
```

## 3. Secret storage — PAM keyring auto-unlock (manual, careful)

`system-setup.sh` enables gnome-keyring and disables KWallet, but it will **not**
edit `/etc/pam.d` for you (a bad edit there can lock you out of login). To make the
keyring unlock automatically with your login password, add these to your display
manager's PAM file — here **`/etc/pam.d/sddm`**:

```
auth       optional     pam_gnome_keyring.so
password   optional     pam_gnome_keyring.so use_authtok
session    optional     pam_gnome_keyring.so auto_start
```

Place `auth` near the other auth lines, `password` after the main `password` line
(`use_authtok` so it reuses the just-entered password), and `session` in the session
block. **Keep a second TTY logged in and test a fresh login before rebooting.**

## What the system layer reproduces

- **NetworkManager only** — `systemd-networkd` + its socket masked, `wait-online`
  disabled. Fixed a 2min→16s boot regression and the "conflicting networks" issue.
- **No-internet false alarm** — re-runs `nmcli networking connectivity check` to
  clear stale `limited` state (IPv4 was always fine; the state was wrong).
- **gnome-keyring** secret storage, **KWallet off**.
- **wifi powersave** drop-in (`/etc/NetworkManager/conf.d/wifi-powersave.conf`).
