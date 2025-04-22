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
Exec=chromium --kiosk http://odoo-test.arandell.com/odoo/barcode
X-GNOME-Autostart-enabled=true

<!-- -disable screen sleep. dont need to do this if done on ui-->

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
<!-- note. there are some issues back in the admin user. The settings in kiosk are supposed to stick over there but. I am seeing some glitches on the admin side -->
<!-- cant get the setting icon on teh OSK to not show. skipping for now. attempting splashtop
    I am able to get into the network via
    "other locations in file explorer

    double clicked: Splashtop_Streamer_Ubuntu_amd64.deb

    I have to install a tool for deb files

 -->

sudo apt install gdebi

<!-- install is good. device shows up in splashtop. but, can not splashtop into it becasue of wayland seession. splashtop does not support wayland. we have to swithc to x11 session  -->

<!-- STARTING OVER ... SPLASHTOP BROKE EVERYTHING -->

<!-- installing timeshift -->

sudo apt install timeshift -y
sudo timeshift --create --comments "Post-GNOME, kiosk user created"

<!-- error here: Failed to create snapshot: Maybe a wrong directory. fixing directory -->

sudo timeshift --gui

<!-- the above gave an error because it was trying to save to the wrong partiiton. also sudo timeshift --gui also failed. below is what worked -->

sudo timeshift --create --comments "Kiosk user created" --snapshot-device /dev/dm-0

<!-- MOVING ON WITH KIOSK INSTALLATION -->

sudo nano /etc/gdm3/custom.conf

<!-- create dir and file if does not already exist -->

mkdir -p ~/.config/AutoStart
nano ~/.config/autostart/chromium-kiosk.desktop

<!-- had an issue here. I forgot to install chromium -->

sudo snap install chromium

sudo timeshift --create --comments "Kiosk user created" --snapshot-device /dev/dm-0

<!-- going to disable settings -->

<!-- clears if from the dock and app launcher. reboot. its still there -->

gsettings set org.gnome.shell favorite-apps "[]"

mkdir -p ~/.local/share/applications
cp /usr/share/applications/org.gnome.Settings.desktop ~/.local/share/applications/
chmod -x ~/.local/share/applications/org.gnome.Settings.desktop

<!-- restore -->

sudo timeshift --restore

<!-- had issues with ttrying to just disable settings for kiosk user. but could not get it to stick. just did it globally -->

sudo mv /usr/share/applications/org.gnome.Settings.desktop /usr/share/applications/org.gnome.Settings.desktop.bak

sudo timeshift --create --comments "settings disable success" --snapshot-device /dev/dm-0

<!-- attempting install for extension to get rid of app icon 9dots -->
<!-- ok followed the above notes. went kind of smoothly. it works -->

sudo timeshift --create --comments "app icon disable success" --snapshot-device /dev/dm-0
exit

<!-- TODO: MAYBE LOCKDOWN CHROME ACCESS AND KEYBOARD SETTINGS ACCESS
 -->

 <!-- BEGINING CLONEZILLA PROCESS. -->

https://clonezilla.org/downloads/download.php?branch=stable

 <!-- CPU Architecture: amd64, File Type: ISO -->
