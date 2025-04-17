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

<!-- extension installed correctly, but due to the server version, have to install the extension to chromium -follow prompts- -->

https://extensions.gnome.org/

<!-- then -->

sudo apt install chrome-gnome-shell gnome-browser-connector

<!-- having issues installing dash to dock. trying to clone it from git -->

git clone https://github.com/micheleg/dash-to-dock.git
sudo apt install make gettext
cd dash-to-dock
make
make install

<!-- ran  into an issue here. had to download -->

sudo apt install sassc
make clean
make
make install

<!-- I think install is in the wrong folder trying to fix this. -->

gnome-extensions enable dash-to-dock@micxgx.gmail.com

<!-- this cmd below was unnecesarry I believe -->

glib-compile-schemas ~/.local/share/gnome-shell/extensions/dash-to-dock@micxgx.gmail.com/schemas

<!-- disable the app icon: 9 dots -->

GSETTINGS_SCHEMA_DIR=~/.local/share/gnome-shell/extensions/dash-to-dock@micxgx.gmail.com/schemas gsettings set org.gnome.shell.extensions.dash-to-dock show-show-apps-button false

<!-- OK FROM HERE. IT LOOKS SATISFACTORY. I just need to take the settings icon out of the top right and keyboard. splashtop -->

<!-- cant get the setting icon on teh OSK to not show. skipping for now. attempting splashtop
    I am able to get into the network via
    "other locations in file explorer

    double clicked: Splashtop_Streamer_Ubuntu_amd64.deb

    I have to install a tool for deb files

 -->
