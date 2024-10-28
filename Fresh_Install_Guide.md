Ubuntu 24.04 fresh install guide
==

First of all, I must say that I'am not a fan or user of Ubuntu. This is just a guide for newcomers, and colleagues that are struggling after the new release with the system changes.

#### Repositories configurations and system upgrade

Right after fresh install, you must do a system update. Don't delay it, but first, let's configure repositories. Delete the new config for repositories, this can cause problems or new users can get lost configuring it, have friends reporting me that:

```
sudo rm /etc/apt/sources.list.d/ubuntu.sources
```

##### Repositories configuration
```
cat << EOF > /etc/apt/sources.list
deb http://us.archive.ubuntu.com/ubuntu noble main universe multiverse restricted
deb http://us.archive.ubuntu.com/ubuntu noble-updates main universe multiverse restricted
deb http://us.archive.ubuntu.com/ubuntu noble-backports main universe multiverse restricted
deb http://security.ubuntu.com/ubuntu noble-security main universe multiverse restricted
EOF
```
If you use this on a server, you can skip the backports line. Perform the system upgrade:

```
sudo apt update && sudo apt dist-upgrade -y && sudo apt clean
```

#### Base install

##### Base install for normal user

```
sudo apt install -y nano mc dialog rsync git net-tools \
 dnsutils iputils-ping telnet libfuse2t64 synaptic curl \
 htop neofetch bpytop clang cargo libc6-i386 libc6-x32 \
 libu2f-udev samba-common-bin exfat-fuse default-jdk rar \
 wget unrar linux-headers-$(uname -r) linux-headers-generic \
 git gstreamer1.0-vaapi unzip ntfs-3g p7zip gcc make lsof \
 build-essential bzip2 tar xz-utils lzma jq libreoffice \
 apt-transport-https gpg
```
I put on the list **synaptic**, why? Cause some users feels more comfortable using synaptic than AppCenter.

#### Look n' Feel

Your Ubuntu looks ugly, or you don't like it by default? No worries, take a look at [LinuxScoop's Youtube channel](https://www.youtube.com/@linuxscoop/videos). You will find easy ways to tune/tweak your desktop.

#### Datetime

There are two ways:

```
sudo timedatectl set-timezone
```

or:

```
sudo dpkg-reconfigure tzdata 
```

You choose which one is the best.

#### Hostname

##### Set your hostname

```
sudo hostnamectl set-hostname name-for-this-pc
```

##### Adjust hosts file

Your must adjust the name of your pc to your localhost, here:
```
sudo nano /etc/hosts
```

#### User management

##### Add a new user

```
adduser sysadmin
```

Doing tasks without typing the password every single time:

```
sudo su
echo "sysadmin ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
```
##### Add it to sudo group:
```
usermod -aG sudo sysadmin
```
##### Disable root login directly
```
passwd -l root
```

#### Remove Snapd
Snapd is broken on 24.04, by default. Try installing firefox or something related to snapd. You will suffer, guaranteed. Try Flatpak or AppImage, but avoid Snapd. Your mental health will thank you later.

First, you must delete all snapd packages installed on the system. 

To do that, you must list and remove:

```
sudo snap list
sudo snap remove $package
```

Stop it's daemon:

```
sudo systemctl stop snapd
```

And now, "Buckle your seat-belt Dorothy, cause Kansas is going bye-bye"

```
sudo apt purge snapd -y
sudo apt-mark hold snapd
```

To avoid it later on the system:

```
cat << EOF > /etc/apt/preferences.d/nomoresnap.pref
Package: snapd
Pin: release a=*
Pin-Priority: -10
EOF
```

Remove all folders that snapd left behind:

```
sudo rm -rf ~/snap
sudo rm -rf /snap
sudo rm -rf /var/snap
sudo rm -rf /var/lib/snapd
sudo rm -rf /var/cache/snapd/
```

#### Drivers
Generally, GNU/Linux drivers are inside the firmware category, for example:

```
firmware-realtek
firmware-amd-graphics
firmware-iwlwifi
firmware-linux-free
firmware-linux-nonfree
firmware-misc-nonfree
```

Or like Nvidia:

```
nvidia-driver
```

Ubuntu has an utility called **ubuntu-drivers devices** that will help you install any driver you might need, for example:

```
sysadmin@ubu2404tpl:~$ sudo ubuntu-drivers devices
udevadm hwdb is deprecated. Use systemd-hwdb instead.
ERROR:root:aplay command not found
== /sys/devices/pci0000:00/0000:00:0f.0 ==
modalias : pci:v000015ADd00000405sv000015ADsd00000405bc03sc00i00
vendor   : VMware
model    : SVGA II Adapter
manual_install: True
driver   : open-vm-tools-desktop - distro free
```

Now you must type:

```
sysadmin@ubu2404tpl:~$ sudo apt install -y open-vm-tools-desktop
```

Warning: some drivers doesn't exists on Ubuntu repositories. I recommend going to GitHub and based on forks/stars select the one according, or the one that fit to your needs, WiFi specially.

#### Installing apps

##### Installing Firefox

Add repo:
```
sudo add-apt-repository ppa:mozillateam/ppa
```

Edit preferences:
```
cat << EOF > /etc/apt/preferences.d/mozilla.pref
Package: *
Pin: release o=LP-PPA-mozillateam
Pin-Priority: 1001

Package: firefox
Pin: version 1:1snap*
Pin-Priority: -1

Package: thunderbird
Pin: version 2:1snap*
Pin-Priority: -1
EOF
```

Update and install:
```
sudo apt update && sudo apt install -y firefox
```

##### Installing Thunderbird

Following the guide for Firefox, just type:

```
sudo apt update && sudo apt install -y thunderbird
```

and you're done.

##### Installing Google-Chrome:

Obtaining the key:

```
wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
```

Sources.list.d line:

```
sudo su
echo "deb https://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/chrome.list
```

And now:

```
sudo apt update && sudo apt install -y google-chrome-stable
```
##### Installing VSCode

Obtaining the key:
```
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > /etc/apt/keyrings/packages.microsoft.gpg
```
Sources.list.d line:
```
echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" | sudo tee /etc/apt/sources.list.d/vscode.list
```
And now:
```
sudo apt update && sudo apt install -y code
```

##### Install VMWare-Pro

From the guys at LinuxScoop, they [released a video](https://www.youtube.com/watch?v=IgskJ0ZjMw4) on this topic.

Download line:
```
wget -c https://softwareupdate.vmware.com/cds/vmw-desktop/ws/17.5.2/23775571/linux/core/VMware-Workstation-17.5.2-23775571.x86_64.bundle.tar
tar -xvf VMware-Workstation-17.5.2-23775571.x86_64.bundle.tar
chmod +x VMware-Workstation-17.5.2-23775571.x86_64.bundle
sudo ./VMware-Workstation-17.5.2-23775571.x86_64.bundle
```
I just want to add, that, if you're using kernel 6.9 or superior(Ubuntu 24.04 has v6.8), you must do the following:

Download, like in the the video:
```
https://github.com/mkubecek/vmware-host-modules/tree/workstation-17.5.1
```
Unzip and modify:
```
unzip vmware-host-modules-workstation-17.5.1.zip
cd vmware-host-modules-workstation-17.5.1/
```
Edit **vmnet-only/vmnetInt.h** and comment out this two lines:
```
#define dev_lock_list()    read_lock(&dev_base_lock)
#define dev_unlock_list()  read_unlock(&dev_base_lock)
```
**Note**: Comments in C are double slashes (//).

And add:

```
#if LINUX_VERSION_CODE >= KERNEL_VERSION(6, 9, 0)
#   define dev_lock_list()    rcu_read_lock()
#   define dev_unlock_list()  rcu_read_unlock()
#else
#   define dev_lock_list()    read_lock(&dev_base_lock)
#   define dev_unlock_list()  read_unlock(&dev_base_lock)
#endif
```
cd ..

Compile the module:
```
make
make install
```
And restart VMWare:
```
/etc/init.d/vmware restart
```
Aaaaand it's done!

##### Spotify Client

Obtaining the key:
```
wget -qO - https://download.spotify.com/debian/pubkey_6224F9941A8AA6D1.gpg | gpg --dearmor | sudo tee /etc/apt/keyrings/spotify.gpg
```
Add sources:
```
cat << EOF > /etc/apt/sources.list.d/spotify.sources
Types: deb
URIs: http://repository.spotify.com
Suites: stable
Components: non-free
Signed-By: /etc/apt/keyrings/spotify.gpg
EOF
```
Installing:
```
sudo apt update && sudo apt install -y spotify-client
```
And enjoy!

##### Installing a DEB app

Tired using:

```
sudo apt install install -f ./another-cool-app.deb
```

You can add GDebi. It will allow you to install a deb file easily:

```
sudo apt install gdebi
```
##### Installing Discord Client

First, [download Discord](https://discord.com/api/download?platform=linux&format=deb)

Following earlier explanation:

```
sudo apt install -f ./discord-*.deb
```
or:

After downloaded, just right click, open/install with gdebi, provide sudo password and install.

#### Codecs

```
sudo apt install -y ubuntu-restricted-extras
```
#### SSH Server

##### Install

```
sudo apt install -y openssh-server
```
##### Copy key from client to server

```
ssh-copy-id sysadmin@server-ip
```
##### Switch to key based auth

Edit the config file:

```
sudo nano /etc/ssh/sshd_config
```

Add/modify these attributes

```
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
ChallengeResponseAuthentication no
```
Restart ssh to apply changes:

```
systemctl restart ssh
```

#### Need Docker?
Easy.
##### Install Docker

```
sudo apt install -y docker.io
```

To use it as a normal user:

```
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

##### Docker-Compose

```
sudo wget -c https://github.com/docker/compose/releases/download/v2.27.1/docker-compose-`uname -s`-`uname -m` -O /usr/local/bin/docker-compose; sudo chmod +x /usr/local/bin/docker-compose
```

A more in-depth guide, [just read here](https://gist.github.com/Koratsuki/cb4e065e8fe7ad3ea3cf34df9bd25c94).

#### Play audio and video files
##### Install
```
sudo apt install vlc rhythmbox
```

#### System fonts in Ubuntu

```
apt install -y fonts-beng fonts-beng-extra fonts-dejavu-core \
 fonts-deva fonts-deva-extra fonts-droid-fallback \
 fonts-font-awesome fonts-freefont-ttf fonts-gargi \
 fonts-gubbi fonts-gujr fonts-gujr-extra fonts-guru \
 fonts-guru-extra fonts-indic fonts-kacst-one fonts-kalapi \
 fonts-knda fonts-lao fonts-liberation fonts-liberation \
 fonts-lklug-sinhala fonts-lohit-beng-assamese \
 fonts-lohit-beng-bengali fonts-lohit-deva fonts-lohit-gujr \
 fonts-lohit-guru fonts-lohit-knda fonts-lohit-mlym \
 fonts-lohit-orya fonts-lohit-taml fonts-lohit-taml-classical \
 fonts-lohit-telu fonts-mathjax fonts-mlym fonts-nakula \
 fonts-navilu fonts-noto-cjk fonts-noto-color-emoji \
 fonts-noto-mono fonts-opensymbol fonts-orya fonts-orya-extra \
 fonts-pagul fonts-pagul fonts-quicksand fonts-sahadeva \
 fonts-samyak-deva fonts-samyak-gujr fonts-samyak-mlym \
 fonts-samyak-taml fonts-sarai fonts-sil-abyssinica \
 fonts-sil-padauk fonts-smc fonts-smc-anjalioldlipi \
 fonts-smc-chilanka fonts-smc-dyuthi fonts-smc-gayathri \
 fonts-smc-karumbi fonts-smc-keraleeyam fonts-smc-manjari \
 fonts-smc-meera fonts-smc-rachana fonts-smc-raghumalayalamsans \
 fonts-smc-suruma fonts-smc-uroob fonts-taml fonts-telu \
 fonts-telu-extra fonts-teluguvijayam fonts-thai-tlwg \
 fonts-tibetan-machine fonts-tlwg-garuda fonts-tlwg-garuda-ttf \
 fonts-tlwg-kinnari fonts-tlwg-kinnari-ttf fonts-tlwg-laksaman \
 fonts-tlwg-laksaman-ttf fonts-tlwg-loma fonts-tlwg-loma-ttf \
 fonts-tlwg-mono fonts-tlwg-mono-ttf fonts-tlwg-norasi \
 fonts-tlwg-norasi-ttf fonts-tlwg-purisa fonts-tlwg-purisa-ttf \
 fonts-tlwg-sawasdee fonts-tlwg-sawasdee-ttf \
 fonts-tlwg-typewriter fonts-tlwg-typewriter-ttf fonts-tlwg-typist \
 fonts-tlwg-typist-ttf fonts-tlwg-typo fonts-tlwg-typo-ttf \
 fonts-tlwg-umpush fonts-tlwg-umpush-ttf fonts-tlwg-waree \
 fonts-tlwg-waree-ttf fonts-ubuntu fonts-urw-base35 \
 fonts-yrsa-rasa fonts-powerline fonts-firacode
```


#### So far so good

Will add more notes later. Stay tuned.
