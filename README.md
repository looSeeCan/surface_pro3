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
 <!-- used belenaEtccher to clone. Was weary about this, because I made an attempt with belenaEtcher in the past and it was glitch.
    seems like it worked successfully now though.
    I need to usbs for this process. The surface only has one usb port. I could not find a functioning usb hub, so I had to work around by
    partitioning the same usb. A few extra steps was involed, including downloading 7zip and manually extracting the iso to the clonezilla 
    partition, becasue belenaEtcher will clone the whole stick and not jsut the partiition.

    ok. clone looks successfull. Checking to see what kind of post edits I need to do.
    
  -->
  <!-- disable sign in a suspend -->

gsettings set org.gnome.desktop.screensaver ubuntu-lock-on-suspend false
gsettings set org.gnome.desktop.screensaver lock-enabled false

<!-- kiosk user can still access settings thru: Power Mode, Bluetooth.
    if I turn all this off can I still connect to bluetooth
 -->

sudo apt remove gnome-control-center

 <!-- the above works. but choosing to just leave this as a post set up. After bluetooth install and any other necessary installs. I can shut it down. -->
 <!-- I have a wifi issue. The wifi icon has been missing. IDK when it started to be missing. This maybe an issue and I need it to come back -->
 <!-- So the icon is always missing. I only have wifi because of the initial ubuntu server install. It connects to the specified wifi that I set theere and 
    then everything gets cloned over. I can no longer access wifi with out makiing drastic tweaks.
  -->
<!-- ATTENMPTING TO FIX -->

sudo apt install network-manager -y
ls /etc/netplan/
sudo nano /etc/netplan/50-cloud-init.yaml

<!-- add the "renderer" key value pair to the .yaml file -->

network:
version: 2
renderer: NetworkManager

sudo netplan apply
sudo systemctl restart NetworkManager

<!-- check now if NetworkManager is managing WiFi -->

nmcli device status
nmcli device wifi list

<!-- I can controll wifi on cli, but it does look like the icon has appeared now. This is satisfactory. -->
<!-- create a timeshift here. -->

sudo timeshift --create --comments "fixed the wifi issue" --snapshot-device /dev/dm-0

<!-- clonezilla process -->
<!-- selecting options on the clonezilla process -->

device-image > local_dev > preffered_location > CZ_IMG > Begginner_accept_the_default_options > savedisk > zip

<!-- selecting options for eh actual clone on a device -->

device-image > local_dev > preffered_location > img_that_was_made_above > begginner > restoredisk > img_that_was_made_above > use_partition_table_from_img

<!-- note the above steps do have options inbetween. the steps listed are the important ones -->

<!-- POST CLONE CHECKS-->
<!-- making sure that there are no ways that the user has to login. If need to switch to admin, can just turn this off. or ssh.-->

gsettings set org.gnome.desktop.lockdown disable-log-out true
gsettings set org.gnome.desktop.lockdown disable-user-switching true

<!-- the above are optional. going to leave them for now and see what happend durihg testing. just inform user not to log out
    no. we need no log in so there is never a password that needs to be used.
    final: disabled the two above settings
 -->

<!-- disable settings access entirely. timeshift here first -->

sudo timeshift --create --comments "disabled log out and user switching so there is never a prompt for a password to login" --snapshot-device /dev/dm-0

<!-- disable settings -->

sudo apt remove gnome-control-center

<!-- tried to disable the search bar, but it seems like you cant so I had to diable the desktop applicaction icons individually -->

sudo mv /usr/share/applications/nm-connection-editor.desktop /usr/share/applications/nm-connection-editor.desktop.bak
sudo timeshift --create --comments "disabled settings and edited the search bar access to search for applications" --snapshot-device /dev/dm-0

<!-- clonezilla -->
<!-- failed to clone -->
<!-- so something was curupted with the master surface. had to boot into the clonezilla shell and: -->

lsblk
sudo fsck /dev/ubuntu-vg/ubuntu-lv

<!-- realized at the end of the coloning option that you can run this check. -->

<!-- CLONING 10 FOR MG -->
<!-- I have to turn on settings for now, so we can initially tweak settings according to needs: wifi, bluetooth.. etc -->
<!-- turned on manually to all devices cloned and turned back on on Master. -->

sudo apt install gnome-control-center -y

<!-- connected bluetooth barcode scanner. turned settings back off -->

sudo apt remove gnome-control-center

<!-- reconnects fine -->
<!--  -->
<!-- time shift here -->

sudo timeshift --create --comments "cloned 10 for Maple Grove. Figured that I should leave settings on untill after post tweaks are done." --snapshot-device /dev/dm-0

<!-- TODO:
   chromium --kiosk \
  --proxy-server="0.0.0.0:1234" \
  --proxy-bypass-list="your-allowed-site.com,localhost,127.0.0.1"

Update autostart cmd
[Desktop Entry]
Type=Application
Exec=chromium --kiosk https://your-allowed-site.com --proxy-server="0.0.0.0:1234" --proxy-bypass-list="your-allowed-site.com"
Hidden=false
X-GNOME-Autostart-enabled=true
Name=Chromium Kiosk

 -->

<!-- May 5th 2025 -->
<!-- ANSIBLE -->
<!--  -->
<!-- doing this project on Jim T's machine. Its a Windows Enterprise. -->
<!-- Windows enterprise does not support wsl 2 -->

-dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

<!-- go to Microsoft store and download -->

-Ubunto 20.04.6 lts installer
-run and setup username, pwd

apt update && apt upgrade

ansible --version

<!-- Make sure ssh is enabled on devices. On control node generate ssh key -->

-ssh-keygen
-ssh-copy-id arandelluser@10.100.16.236

<!-- Need to add the ssh key to each user that requires controlling. going to default to just adding the kiosk -->

-ssh-copy-id kiosk@10.100.16.236

<!-- Clean mainatainalble inventory -->

- mkdir (check the folder structure in ansible)

<!-- edit the hosts.ini file: -->

-check file

<!-- added another device. had to change the hostname -->

- sudo hostnamectl set-hostname MF-FORK-TB-TEST1
- sudo nano /etc/hosts
- edit hostname accordingly
- running to an issue here where I change the hostname and chromium does not autostart anymore
  <!-- chromium creates a file "singleton" that has to be deleted everytime the hostname is changed or chromium will not start. in this dir: -->
  -~/snap/chromium/common/chromium$
  - sudo rm Singleton\*

<!-- after removing the Singleton files, edit .config/autostart -->

Exec=chromium --kiosk --user-data-dir=/tmp/kiosk-profile http://odoo-test.arandell.com/odoo/barcode

<!-- can now change hostname and chromium will autostart. this opens a temp/disposable profile each time. this does cause sign out to happen everytime a reboot happends -->

sudo hostnamectl set-hostname example

<!-- timeshift here -->

sudo timeshift --create --comments "made some changes due to issues arising when changing the hostname." --snapshot-device /dev/dm-0

<!-- timeshift in kiosk user is fine, just include /home/kiosk in timeshift settings -->

sudo timeshift --create --comments "included /home/kiosk in timeshift." --snapshot-device /dev/dm-0

<!-- cloning. cloned using the date as the name of the clone -->
<!-- good. tested changing hostnames, chromium is starting as usual.  -->
<!-- continue with ansible -->

<!-- copied ssh key to devices: .8 and .230. I can ssh without password -->

<!-- May 7th 2025 -->
<!-- testing -->

ansible -i inventory/hosts.ini kiosks -m ping

<!-- the above did not work. Ran into an error:
The error was: ModuleNotFoundError: No module named 'ansible.module_utils.six.moves'
 -->

<!-- On the Control Node:
note that I did install on the devices also
 -->

<!-- Updating. So a combination of an ansible bug, which needed upgrading and the directory needed to be fixed -->

sudo apt install python3-pip
pip3 install --user --upgrade ansible
echo 'export PATH=$HOME/.local/bin=$PATH >> ~/.bashrc source ~/.bashrc'

<!-- It seems like I am having this issue becasue I am installing ansible as the user. I need to install ansible globally
 -->

pip3 uninstall ansible
install globally
sudo pip3 install --upgrade ansible

confirm
/usr/local/bin

<!-- this now works -->

ansible -i inventory/hosts.ini kiosks -m ping

<!-- Before moving on, I want to test if on a new cloned device i have to install python manually like i did .
  for the ohter devices
-->

-clone device
-test if it will ping: ansible -i inventory/hosts.ini kiosks -m ping
-it should fail because the python update that I did on the initial two devices are not on this one:

    -cloned device: good
    -changed hostname: chromium still autostarts
    -ssh-copy-id kiosk@10.100.16.6
    -ssh without password: good

    <!-- so the above did not fail when I ansible pinged it. the update on the controll node was good enough. -->

<!-- test playbook. update all devices -->

<!-- create a new file in kiosk_project -->

nano update_all.yml

<!-- check the yaml file -->

<!-- remember to be in the right dir -->

ansible-playbook -i hosts.ini ../update_all.yml

<!-- I have an error here about sudo.
  sudo for kiosks uses a password. the "become: true" in the playbook says to run the comman with sudo. sudo needs a password
 -->

ansible-playbook -i hosts.ini ../update_all.yml --ask-become-pass

 <!-- ok everything checks out from here. I created a playbook that updates and upgrades all device. 
 
 Todd: -complete- Moving forward with this I need to make sure dns and dhcp is correct
  for these devices
  -->

<!-- MAY 8TH 2025 -->
<!-- Steve has requested 3 devices for autocount. I have three ready. Probably just have to switch autostart to autocount.arandell.com. -->
<!-- made apporpriate changes to the three devices:
  hostnamectl, edited hosts.ini
  ansible ping worked
  when running update playbook, ran into an error:
   -something about: cant fine the efi system partition. I did not have an issue with the effeted devices prior. Im not sure if a reboot did this or not.

 -->
 <!-- fixed the above on the effected devices-->

sudo dpkg --configure -a
sudo apt install -f

<!-- ^solved the issue for now. rebooted and confirmed -->

 <!-- AUTOSTART autocount -->
 <!--edited autostart file to this-->

Exec=chromium --kiosk --user-data-dir=/tmp/kiosk-profile http://autocount.arandell.com

<!-- works but we need it to be able to open tabs when
  we click on the links. from the setup on odoo, the tabs and the search bar is disabled for the chromium browser
 -->

 <!-- had to troubleshoot the ^ above with a couple things
    when i added the teh new Exec line. it gave me the sigleton error again. deleted the singleton files then added :
      -->

      Exec=chromium --user-data-dir=/tmp/kiosk-profile http://autocount.arandell.com

<!-- tnis one^ eventually autostarts with chromium showning all the tabs. this is satisfactory. will see what kind of constraints we need after testing -->
<!-- actually its not satisfactory. the user will be able to close chromium. you can search and reopen chromium but the config file set it to where chromium is a
  temp and it will not let you back into the url
  im going to try to  make the profile a persistent profile instead of a temp, but I remember I made it a temp for a reason
 -->

--user-data-dir=/home/kiosk/.kiosk-profile

<!-- still acting the same I think its because I change the Exec, I have to delete the singelton files again-->
<!-- IM AN IDIOT. THE ISSUE WAS THAT IT WAS AUTOMATICALLY "https" -->

<!-- revert to  -->
<!-- reverted back to this config and works good. had to rm the singleton files again to make chrome open up again after changing the config file
  seems like I have to do the singleton thing everytime I change the config file
  also got that error agin with the autoupdate playbook. I think it has to do with the singleton also... think anyways.
 -->

Exec=chromium --user-data-dir=/tmp/kiosk-profile http://autocount.arandell.com

 <!-- May 21 2025 -->
<!-- cloning a couple more for Corvin. -->
<!-- we need to install vnc: Tight vnc with Xfce(lightweight desktop environment) -->

sudo apt update
sudo apt install xfce4 xfce4-goodies -y

<!-- got a purple config screen about the display manager. press ok
  selct: lightdm
   -->

  <!-- disable display manager immediately -->

sudo systemctl disable lightdm
sudo systemctl stop lightdm

<!-- the above hs broken everything. abort -->

<!-- JUNE 5TH 2025
  Problem:
  There is an odoo issue. I do not believe it is relaed to the tablet.
 -->
 <!-- making some edits to ansible before i move on
   - had to make some edits to host.ini and update_all.yml
   - creating and adding these devices to ansible. had to do some troubleshooing for update to work. something like the prior
     update was interupted, had to: sudo dpkg --configure -a -->

 <!-- only updates [kiosks_mg] -->

ansible-playbook -i inventory/hosts.ini update_all.yml --limit kiosks_mg --ask-become-pass

<!-- Bluetooth issue - bluetooth issue has been reported. it is dropping out with certain situations. When screeen is goes "black". I am assuming the power settings need to adjust to "never" sleep and the physical power button set to "nothing". -->

  <!-- to see the bluetooth devices on kiosk -->

bluetoothctl

   <!-- I have to use 123Scan to edit the zebra bluetooth name. -->
   <!-- question:
     - is the zebra scanner and the dock specific to each other? Do they need to be together?
     - can the cradle be hooked up to the kiosk > scanner no bluetooth to the kiosk, so we can bypass the blutooth situation or
      do we require bluetooth
       - yes. the hookup to the kiosk does work. it scans. will it charge it though
    -->

<!-- note: hostname in /etc/hosts might be causing an some issues
  we changed some /etc/hosts files. we will see if that works out. It did work on Ray33
-->

<!-- I did not use 123Scan for naming the scan guns. ended up just scanning the letters by hand. -->

<!-- June 6th 2025 -->
<!-- checking system logs -->

journalctl -u NetworkManager --since "2 hours ago"

<!-- ok so the time stamp on the log was not correct. after some investigation. it looks like the date and time settings is set to: UTC London -on the UI-. WTF! This maybe an issue. Fix this:-->

timedatectl
sudo timedatectl set-timezone America/Chicago

<!-- for live monitoring -->

sudo journalctl -u NetworkManager -f

<!-- I've went and reconfigured all the devices that are currently running: 18,22,29,37,ray16,ray33
  power options, datetime, hostname, labeling bluetooth.

  I think maybe the .yaml file might be conflicting with connections.
  36 is acting up. bluetooth is not working. chromium wont stratup. so I am going to timeshift 36 and configure from there.
  starting with getting rid of the netplan issue to see if it will resolve the issue.

  Im going to set it up on this newly cloned device and test
   -so there is something going on with the way netplan and NetworkManger is working.
   gotta do more research. Im not sure if I can just keep the netplan file or do I hav to go away from it.
   just delete all the yaml files except for the approprite one: Test this on one machine.

 -->

<!-- June 9 2025 -->

<!-- wip18 /////////////////////////////////////////////////
 is losing connection. All settings seem to check out at this point: power saving opitons, bluetooth, appropriate .yaml file
 I am attempting to to edit some power management on the network manager. Wifi power save
-->
<!-- have to install -->

iwconfig

<!-- after installation iwconfig will show:
  IEEE 802.11  ESSID:"Your-WiFi-Network"
          Mode:Managed  Frequency:5.24 GHz  Access Point: XX:XX:XX:XX:XX:XX
          Bit Rate=450 Mb/s   Tx-Power=22 dBm
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Encryption key:off
          Power Management:on  <-- THIS IS THE SETTING
          Link Quality=70/70  Signal level=-40 dBm
          Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
          Tx excessive retries:0  Invalid misc:0   Missed beacon:0

  I have to turn the "Power Management" off
 -->

 <!-- shows power save status -->

iw dev wlp1s0 get power_save

 <!-- there was a default configuration file: /etc/NetworkManager/conf.d/default-wifi-powersave-on.conf
   I changed the:
   wifi.powersave = 3
   to
   wifi.powersave = 2

   that turns it off
   sudo systemctl restart NetworkManager

   the power management is off

   So, editing the default config file caused somehing to break. the screen went black, but did not sleep. it just goes black. you can turn it
   back on, but goes black again.
   reverted the default-wifi-powersave-on.conf 
   created a config file in the same dir. 
   default-wifi-powersave-on.conf.bak

   the new file 99-......
   now reflects that the "Power Management" is off. the screen is not doing that anymore
   Its out there testing, but the log files show the same issue... 
  -->

  <!-- right before I left wip18s connection started to slow down, with a loading on bottom right of odoo window. it looks like a very slow connection,
  it has to be device specific, because wip34 was right next to it and i did speed tests on both and wip34 had a fast connection
  rebooted wip18 and the speed test was good again.
  could this be cache related, specic to the user kiosk autostart? or to eh nework settings I have been working on?
  explore canceling NetworkManager and just using default. I remember I setup NetworkManager because my thought was that they needed a UI to connect to a network. June 10th 2025
  -->
  <!-- june11th -remember that this date here is s a category under wip18 
    came in this morning and wip18 was damaged. replaced it with a windows os device to see if the error reporoduces there.
    I am going to clone this device as is and use it for 36 also.
  -->
  <!-- so I replaced this device with a windows os device and it ran all day, but at the end of the day here approximately 4:40 it started to do the exact same thing as last night -check prior note^  
    noticed that datetime was off on this one. fixed it and the speed test seems to be good. will check tomorow.
  -->

<!-- wip34 /////////////////////////////////////////////////////////////////////////////////////////////////
 I found some errors on this device. I am assuming its the same errors thats going on with wip18
   edited so the Power save: off
 -->

   <!-- and created and edited this file: /etc/modprobe.d/mwifiex_pcie.conf
   added thsi line to that file: options mwifiex_pcie ps_enable=0 -->

    sudo nano /etc/modprobe.d/mwifiex_pcie.conf
    sudo update-initramfs -u

<!-- wip22 ///////////////////////////////////////////////////////////////////////////////////////////////////
  this device is actiually loosing the UI wifi toggle switch
 -->
 <!-- list the pci devices -->

lspci -k

<!-- update firmware -->
<!-- june 11th -->

sudo apt update
sudo apt install --reinstall linux-firmware
sudo update-initramfs -u
sudo reboot

<!-- put it back on the truck and test after the above firmware updates -->
<!-- it failed pretty fast. found a log that maybe a confilicting issue with netplan and networkmanager. going thru steps releive this -->
<!-- remove .yaml file -->

sudo systemctl stop NetworkManager
sudo rm /etc/netplan/...yaml /etc/netplan/....yaml

<!-- rm any /system-connections files. there are none in here -->

sudo rm /etc/NetworkManager/system-connections/

<!-- start networknamager -->

sudo systemctl start NetworkManager

<!-- this is not working. tried many routes. NetworkManager is stil creating a .yaml file in netplan.
  Asked on reddit, waiting for reply
  configured:
  IEEE 802.11  ESSID:"Your-WiFi-Network"
          Mode:Managed  Frequency:5.24 GHz  Access Point: XX:XX:XX:XX:XX:XX
          Bit Rate=450 Mb/s   Tx-Power=22 dBm
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Encryption key:off
          Power Management:on  <-- THIS IS THE SETTING
          Link Quality=70/70  Signal level=-40 dBm
          Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
          Tx excessive retries:0  Invalid misc:0   Missed beacon:0

 -->
 <!-- back on truck -->

<!-- update kernel -->

<!-- JUNE 12TH 2025///////////////////////////////////////////////////////////////////////////////////////////// -->
<!-- ray16
 made my rounds and went to check out this device. This truck has been idle in the same spot for a couple days.
 I took the device on a stroll with me to check in on the ohter devices. didnt drop out. Performed an update and upgrade at this point. note that the turrets barely have amy changes on them accept for
 powersave and datetime.

 Lance says that besides the network related disconnect over night, everything was good, except for wip18,
 which I know about. That is a windows device that is not autostartign, so I just had to open the browser.
 All 5 devices on the floor are on line. I am working on a device with all the configuration edits that I mades
 since I have been down here and clone them to replace 18 and 36 and more if possible to leave corvin with some extra devices
 -->

 <!-- Plan:
   make/check the following edits/configuratiions that I have made on the devices since I have been here at MG:

   power options, location -note: location is not turned on on the other devices-, datetime, hostnamectl, hosts, autostart, netplan, wifi power management
   dont forget to rm Singleton files when changing autostart configuration/or any other that would casues chromium not to autostart
   bluetooth is good
   Cloning device:
   reserve the ip and mac of master prior to cloning
   check first clone to make sure macs and ips are diff
   ^good

   cloned new deviece
   replacing mg-fork-tb-wip18 -was damaged-
   removed mg-fork-tb-wip18 from dhcp
   reserve ip and perform Singleton
   had to do some updates

   cloned new device
   mg-fork-tb-wip36
   performed the clone on the same device. -this device had some uniknown errors- make sure to remove from dhcp first
   bluetooth is not working on this device
   cloned a surface pro 5. new issues have arisen. no rotation and touch screen is off

  below is the original setting from the original clone that I sent to MG. Will explore this setting if issues arise
  Exec=chromium --kiosk --user-data-dir=/tmp/kiosk-profile http://odoo.arandell.com/odoo/barcode

  -->

<!-- wip18
 scanned pallets are not reflecting in odoo. This maybe an odoo issue on the wip18 sign in?
 DISREGARD THIS. User error. Todd was looking at the wrong category. This is working. Still need to replace with ubuntu device though

 -->
<!-- Surface Pro 5////////////////////////////////// -->
<!-- New issue has arisen when attempting to clone to a Surface pro 5
  touch screen and rotation not functional. only functional with keyboard attached. looks like i may need a kernal for pro5.
  Attempting:
 -->
 <!-- Add the GPG key -->

wget -qO - https://raw.githubusercontent.com/linux-surface/linux-surface/master/pkg/keys/surface.asc \
 | gpg --dearmor \
 | sudo tee /etc/apt/trusted.gpg.d/linux-surface.gpg >/dev/null

  <!-- Add the repo -->

echo "deb [arch=amd64] https://pkg.surfacelinux.com/debian release main" \
 | sudo tee /etc/apt/sources.list.d/linux-surface.list

<!-- update and install  -->

sudo apt update
sudo apt install linux-image-surface linux-headers-surface iptsd libwacom-surface

<!-- ok. this ended up going to the black GNU screen with options. When choosing the option "ubuntu": error: shim signature
  I have to go and turn off sucure boot in bootloader.
  After that it worked.. initially at least
  double check the edits/configs.. some power options were still on
 -->

<!-- wip29 //////////////////////////////////////////////
 approximately 4:05 wip29 is lost connection. It was going pretty strong. note: that there are very limited edits to this device. It was not being operated most of the time I was here.
 this maybe good. will investigate tomorrow before I leave


<!-- JUNE 13 2025//////////////////////////////////// -->

<!-- wip29 -->
<!--
 editing:
 netplan
 loacation
 install updates
 -->

 <!-- identify: wifi harware, current kernel version, Os  -->

lspci -nnk | grep -i net -A3
uname -r
lsb_release -a

<!-- updating firmware -->

sudo apt update
sudo apt install --reinstall linux-firmware
sudo reboot

<!-- edited power management -->

iwconfig

<!-- trie to copy a clonezilla usb for Corvin, which would have the images of the surface pro 5 and surface pro 3, but am running into some complications
 manually cloning the rest of his pro 5's for him before I leave

 steps:
 turn off secure boot, then use above cmds to install kernal


set up complete.
 -->

<!-- JUNE 17TH 2025
  setting up devices to prepare for "testing sessions" per ticket from Trudy.
  cloning last img from Maple Grove: mg-fork-tb-master

  before clone, make sure surface is up to date.
   - noticed here that the surface 3s are not able to connect to any network, but when I clone them it connects fine

  netplan, update and upgrade, hostnamectl, singleton files, dhcp, ansible

  TODO: just need to name bluetooth device
 -->

<!-- JUNE 19TH 2025
  as issue has arisen with the surface pro5 setup. When the odoo web app crashes -most likely due to the underlying network issue- the surface boots to UEFI.
  all booting config is correct. I tride to reproduce the error by going outside with an exact clone to replicate the low wifi signal. I can get the web app to crash but when I reboot I cant replicate the UEFI screen. according to Mitch, The UEFI screen is present and when he takes it to his office it boots directly to where it is supposed to, which is the autostart oddo page.
  I think it has to do with the underlying network issue.

  On another note. I was able to configure RDP. Turns out the setup that I have includes vnc and all I have to do is turn it on o the UI. I did and I had Corvin turn some on on his side. looks good
   now when looking at one of the problematic pro5s I do see that the "powersave" option is tuned on. This explains why it is sleeping when I was ssh'd into it. but does it explain the UEFI issue?

 -->

<!-- JULY 2 2025 -->
  <!-- Setting up some more for MG. Corvin still has four available. 
    does not need much post set up here. the clone is configured from the last time I visited. Just turn on remote desktop.
  
  -->

<!-- JULY 7TH 2025
  On site here at Maple Grove. Made my rounds to check configurations on tablets. Nothing out of the ordinary of whats been going on. Wifi has been worked on and it seems like it is supposed to be working.
  Ideally, the devices should not be dropping out anymore.

  Next day:
   35 is having drop issues. It was an old device that was first shippe out. thought that was the issue, but it seems like there are still issues with drops
   I replaced 35 with an unamed device to see if the drops are still happening.

   Replaced the old 35 device with a new one. confifigurations checked:
    netplan, update and upgrade, hostnamectl, singleton files, dhcp, ansible

    5ghz
     confirmed that there is a 5ghz connection. trying to connect to it atm. I might have to go to the floor in the morning and try to make a connection.

  Next day:
    I noticed that there were two devices with the host name wip34, though they had different ips. Alieviated that and made sure the right one was connected to the truck

    Setting up 4 more devices and I noticed that the netplan file is the same netplan file being cloned over. I need to rm the file and connect via NetworkManager so each devices netplan file is unique.

  Next day -friday-
    turr16 had an issue. first time that that device had any issue. I did very minimal adjustments to the turr and they have been working fine. I think it's because they are static.
    updating the master device here: didnt finish this

    there is a problem with the 4 devices that I brought over. I cant get the kernal to work that fixes the touch screen. attempting to clone one that already works
 -->

<!-- JULY 18TH 2025
  So we need to setup 12 devices for Menomonee Falls by Mondy. I am continuing to see the problem with the surface 5's -touch screen and rotation-.
  I successfuuly fixed one here:
    there seems to be an error showing up about a bad file: /etc/apt/sources.list.d/linux-surface.listecho
    I removed this file and re-added the repo properly:
 -->
<!-- delete file -->

sudo rm /etc/apt/sources.list.d/linux-surface.listecho

<!-- re add the repo properly. I guess I added it wrong somewhere, which then created a file: linux-surface.listecho. its supposed to be: linux-surface.list -->

echo "deb [arch=amd64] https://pkg.surfacelinux.com/debian release main" \
| sudo tee /etc/apt/sources.list.d/linux-surface.list

<!-- I am not sure if this will be the fix that I need with the 4 at MG. I hope so. JUST NEED TO CHECK IF THAT BAD FILE IS THERE. -->
<!-- NEXT STEP HERE AT MF:
  I need to clone this pro5 as a master and prepare the 12 that we need here.
 -->

<!-- update and upgrading may break the surface kernal
  lets test:
    1.clone a surface5 from the latest clone -the surface3-
    2. do not update upgrade
    3. confirm that touchscreen does not work
    4. install the kernal to make it work
    5. conifrm it works
    6. the update upgrade
    7. see if the kernal broke from the upgrade

    check the above:
      1. check
      2. check
      3. check
      4. check
      5. touchscreen does not work. rotion does. note: I just rm the netplan file and reconnected to "plant" so I can ssh.reinstalling iptsd and rebooting worked. touchscreen now works
      6. no, a full update and upgrade did not break this one


-->

<!-- if it does break -->

sudo apt install --reinstall linux-image-surface linux-headers-surface iptsd libwacom-surface

<!-- JULY 22 2025
  It seems like the issue above is working. I need to move on and try to clone a working pro5

  ok. so I have ran into this issue beforen where there is a Partclone error. Steps to repair:
    just select the options to: "interactively check/repair fsck"
    choose option 1 for all questions. and yes to questions after that

  got another error about not enough space. turns out I have to many failed images. deleted all unnecessay ones. Now I just have the pro3 and pro5 images.
  cloning a pro5 right now with the pro5 image that was just created. note that this device was not initially updated in windows os at all to turning on. if successful, toouch screen and rotation should work. will not have to do the whole kernal step.
  Rotation works, but touchscreen did not on intial boot. could not access anything, so I did a hard reset. Rebooted and now touch screen works...hmmmm. this is hwat happened in MG also. I thought that those devices
  were not ready, when I called corvin, the issue resolved itself. They must of ran out of battery, shut down , theh when Corvin rebooted them, the issue resolved....hmmm...

  <!-- TODO: remember to disconnect and reconnect from network so devices have their own unique connection to the network and not the cloned one!!! -->

-->

<!-- JULY 22 2025
  Just cloned a surface pro 5 with same image and everything worked fine: touch screen and rotation. didnt have to reboot like I did above....hmmmm.....

  In continuing with the setup for MF I am opting to not connect to the guns to bluetooth and just connect them to the cradle.
  Bluetooth and wifi use the same bandwidth. so this may free it up... maybe.
 -->
  <!-- Master device -for both surface 3 and 5 master devices:
    IS IT OK TO UPDATE AND HOW OFTEN?
    need to do a snapshot
    do update after snapshot
    if breaks, then go back to snapshot
  -->
  <!-- just used the ui to snapshot. also this is an option
    to make sure the kernal does not break during updates, but so far I have not had to use it. will look into this later
   -->

    sudo apt-mark hold linux-image-surface linux-headers-surface

<!-- TODO: NEED TO CREATE MASTER DEVICE FOR SURFACE PRO 3. Besides that, most of the tablets are ready. I have 11 of them. should be good for now. -->

<!-- AUGUST 6TH 2025
  I have names for the devices. Going thru the 11 that I had setup and naming them.
  changes being made:
  hostnamectl
  hosts
  Singleton
  dhcp
  Label

  completed list that Jeff sent:
    110p
    112P
    435W. Press.
    441W. Bindery.
    475W. Ship 22.
    443W. Ship 23.
    382W. WIP.
    834P. Roll 2.
 -->

<!-- AUGUST 12 2025
  contiuning with an updated list:

 -->

 <!-- ran into an issue with dhcp -->
 <!-- refresh dhcp -->

nmcli device reapply wlan0
