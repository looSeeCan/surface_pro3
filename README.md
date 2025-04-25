# Surface Pro 3/5 Ubuntu GNOME Kiosk Setup

<!-- please refer to rough draft notes for details -->

> _Personal Notes & Final-ish Guide_

A concise, step-by-step reference for converting an Ubuntu Server‑only install on Surface Pro 3/5 devices into a locked‑down GNOME kiosk environment.

---

## 1. Base System

1. **Install Ubuntu Server** (20.04+).
2. **Update & upgrade** packages:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

---

## 2. Minimal GNOME Installation

```bash
sudo apt install -y \
  gnome-session gdm3 gnome-terminal \
  nautilus gnome-control-center
```

- Verifies that GNOME, the display manager (GDM), and basic utilities are installed.

---

## 3. Chromium Kiosk Browser

1. **Install via Snap:**

   ```bash
   sudo snap install chromium
   ```

2. **Add a dedicated kiosk user:**

   ```bash
   sudo adduser kiosk
   sudo usermod -aG sudo kiosk
   ```

3. **Enable auto-login** (for `kiosk`):

   ```bash
   sudo sed -i 's/#\s*AutomaticLoginEnable=false/AutomaticLoginEnable=true/' /etc/gdm3/custom.conf
   sudo sed -i 's/#\s*AutomaticLogin=.*/AutomaticLogin=kiosk/' /etc/gdm3/custom.conf
   ```

4. **Autostart Chromium** (as kiosk):
   ```bash
   sudo -u kiosk mkdir -p /home/kiosk/.config/autostart
   sudo tee /home/kiosk/.config/autostart/chromium-kiosk.desktop << 'EOF'
   [Desktop Entry]
   Type=Application
   Name=Kiosk Browser
   Exec=chromium --kiosk http://odoo-test.arandell.com/odoo/barcode
   X-GNOME-Autostart-enabled=true
   EOF
   ```

---

## 4. Lockdown GNOME

> Disables idle timers, power actions, and hides UI elements.

```bash
# Disable screen sleep + lock
sudo -u kiosk gsettings set org.gnome.desktop.session idle-delay 0
sudo -u kiosk gsettings set \
  org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type 'nothing'
sudo -u kiosk gsettings set \
  org.gnome.settings-daemon.plugins.power sleep-inactive-battery-type 'nothing'

# Hide launcher & apps menu
sudo -u kiosk gsettings set org.gnome.shell favorite-apps "[]"
sudo -u kiosk gsettings set org.gnome.desktop.app-folders.folder-children "[]"

# Disable Settings app globally
sudo mv /usr/share/applications/org.gnome.Settings.desktop{,.bak}
```

---

## 5. GNOME Extensions: Dash‑to‑Dock

```bash
sudo apt install -y git make gettext sassc
cd /tmp
git clone https://github.com/micheleg/dash-to-dock.git
cd dash-to-dock
make && sudo make install
cd ~ && rm -rf /tmp/dash-to-dock

# Enable and customize
sudo -u kiosk gnome-extensions enable dash-to-dock@micxgx.gmail.com
sudo -u kiosk gsettings set \
  org.gnome.shell.extensions.dash-to-dock show-show-apps-button false
```

---

## 6. System Snapshots with Timeshift

```bash
sudo apt install -y timeshift
sudo timeshift --create --comments "Post-GNOME & kiosk setup" --snapshot-device /dev/dm-0
```

- Use `sudo timeshift --restore` to roll back.

---

## 7. Networking & Utilities

- **Install NetworkManager** (to restore Wi‑Fi icon):

  ```bash
  sudo apt install -y network-manager
  sudo sed -i '/^renderer:/d' /etc/netplan/*.yaml
  sudo sed -i 's/version:/version:\n  renderer: NetworkManager/' /etc/netplan/*.yaml
  sudo netplan apply
  sudo systemctl restart NetworkManager
  ```

- **Verify Wi‑Fi**:
  ```bash
  nmcli device status
  nmcli device wifi list
  ```

---

## 8. Clonezilla Workflow (Optional)

1. **Download ISO**: [Clonezilla Stable](https://clonezilla.org/downloads/)
2. **Write to USB**: Use Etcher or manually extract via 7‑Zip.
3. **Run through menus**: Device‑image → local_dev → savedisk/restoredisk → advanced options as needed.
4. **Post‑clone fsck**:
   ```bash
   sudo lsblk
   sudo fsck /dev/ubuntu-vg/ubuntu-lv
   ```

---

## 9. Final Lockdown & Cleanup

```bash
# Prevent logout & user switching
sudo -u kiosk gsettings set org.gnome.desktop.lockdown disable-log-out true
sudo -u kiosk gsettings set org.gnome.desktop.lockdown disable-user-switching true

# Remove Control Center if desired
gnome-control-center safe-uninstall || sudo apt remove -y gnome-control-center
```

---

> _End of personal notes. Copy to a feature branch for iteration._
