Gnome Minimal
-odoo
90i45pTH

installed ubuntu server first

sudo apt install gnome-session gdm3 gnome-terminal nautilus gnome-control-center -y

sudo snap install chromium

sudo adduser kiosk

<!-- -add user to sudo (for setup) -->

sudo usermod -aG sudo kiosk

<!-- -enable autologin for kiosk -->

sudo nano /etc/gdm3/custom.conf

<!-- -edit the custom.conf to reflect below -->

[daemon]
AutomaticLoginEnable = true
AutomaticLogin = kiosk

<!-- -create the autro start folder and file -->

mkdir -p ~/.config/AutoStart
nano ~/.config/autostart/chromium-kiosk.desktop

<!-- -in the file -->

[Desktop Entry]
Type=Application
Name=Kiosk Browser
Exec=chromium --kiosk http://odoo-test.arandell.com/
X-GNOME-Autostart-enabled=true

<!-- -disable screen sleep -->

gsettings set org.gnome.desktop.session idle-delay 0
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type 'nothing'
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-battery-type 'nothing'

<!-- Disable Access to Settings in GNOME. make sure kiosk we are signed into kiosk user. Check above also. those steps may also require to be logged in as kiosk
    this step is important for the cloning step. it makes sure that kiosk user retains all the settings.
 -->

gsettings set org.gnome.shell favorite-apps "[]" - clears if from the dock and app launcher
gsettings set org.gnome.desktop.app-folders.folder-children "[]" -hide all app folders
gsettings set org.gnome.desktop.app-folders.folder-path "[]"
sudo chmod -x /usr/share/applications/org.gnome.Settings.desktop - make the settings app file non-executable

<!-- some of the above did not work to remove the Settings. the one below did. not sure if I even need to do the ones above -->

sudo mv /usr/share/applications/org.gnome.Settings.desktop /usr/share/applications/org.gnome.Settings.desktop.bak

<!-- turning back on temp so I can make adustments to the device. then will turn back off. -->

<!-- installing ext to hide dock -->

sudo apt install gnome-tweaks gnome-shell-extensions -y
