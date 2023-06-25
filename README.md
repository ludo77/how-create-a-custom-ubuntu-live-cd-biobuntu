# how-create-a-custom-ubuntu-live-cd minimal Biobuntu

I use this for create my own version ubuntu: https://itnext.io/how-to-create-a-custom-ubuntu-live-from-scratch-dd3b3f213f81

I use jammy version of ubuntu

1) Prerequisites (GNU/Linux Debian/Ubuntu)

sudo apt-get install binutils debootstrap squashfs-tools xorriso grub-pc-bin grub-efi-amd64-bin mtools
    
mkdir $HOME/live-ubuntu-from-scratch


2) Bootstrap

sudo debootstrap --arch=amd64 --variant=minbase jammy $HOME/live-ubuntu-from-scratch/chroot http://us.archive.ubuntu.com/ubuntu/
 
 
3) Configure external mount points

sudo mount --bind /dev $HOME/live-ubuntu-from-scratch/chroot/dev

sudo mount --bind /run $HOME/live-ubuntu-from-scratch/chroot/run

4) Access chroot environment

sudo chroot $HOME/live-ubuntu-from-scratch/chroot

5) Configure mount points, home and locale

mount none -t proc /proc

mount none -t sysfs /sys

mount none -t devpts /dev/pts

export HOME=/root

export LC_ALL=C

5) Set a custom hostname

echo "biobuntu-live" > /etc/hostname

6) Configure apt sources.list

cat <<EOF > /etc/apt/sources.list

deb http://archive.ubuntu.com/ubuntu/ bionic main restricted universe multiverse 

deb-src http://archive.ubuntu.com/ubuntu/ bionic main restricted universe multiverse

deb http://archive.ubuntu.com/ubuntu/ bionic-security main restricted universe multiverse 

deb-src http://archive.ubuntu.com/ubuntu/ bionic-security main restricted universe multiverse

deb http://archive.ubuntu.com/ubuntu/ bionic-updates main restricted universe multiverse 

deb-src http://archive.ubuntu.com/ubuntu/ bionic-updates main restricted universe multiverse 

EOF

7) update packages

apt-get update

8) Install systemd

apt-get install -y systemd-sysv

9) Configure machine-id and divert

dbus-uuidgen > /etc/machine-id

ln -fs /etc/machine-id /var/lib/dbus/machine-id

dpkg-divert --local --rename --add /sbin/initctl

ln -s /bin/true /sbin/initctl

10) Install packages for Live System

apt-get install -y ubuntu-standard casper lupin-casper discover laptop-detect os-prober network-manager resolvconf net-tools wireless-tools wpagui locales linux-generic
    
11) Graphical installer

apt-get install -y  ubiquity  ubiquity-casper ubiquity-frontend-gtk ubiquity-slideshow-ubuntu ubiquity-ubuntu-artwork

12) install lightdm (simple display manager), gnome-session and gnome-terminal (minimal gnome), xfonts-base(standard fonts for X) ,xserver-xorg(X.Org X server) xinit(X server initialisation tool) firefox(Mozilla Firefox web browser) thunderbird(mail/news client with RSS, chat and integrated spam filter support) nautilus(file manager and graphical shell for GNOME) libreoffice(office productivity suite) gimp(GNU Image Manipulation Program)

apt install lightdm gnome-session gnome-terminal xfonts-base xserver-xorg xinit thunderbird nautilus libreoffice gimp imagej vlc python3-biopython
pip install jupyterlab

13) install this (for install ppa)

apt-get install software-properties-common

14) install ugene(integrated bioinformatics toolkit)

apt install ugene

15) install zotero (organize and share your research sources)

wget "https://www.zotero.org/download/client/dl?channel=release&platform=linux-x86_64" -O zotero64.tar.bz2
tar -xf zotero64.tar.bz2
mv Zotero_linux-x86_64 /opt/zotero
cd /opt/zotero
./set_launcher_icon

16) install R (GNU R statistical computation and graphics system)

cd /
wget -qO- https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc | gpg --dearmor -o /usr/share/keyrings/r-project.gpg
echo "deb [signed-by=/usr/share/keyrings/r-project.gpg] https://cloud.r-project.org/bin/linux/ubuntu jammy-cran40/" | tee -a /etc/apt/sources.list.d/r-project.list
apt install --no-install-recommends r-base r-base-dev

17) install RStudio (integrated development environment (IDE) for R)

wget https://download1.rstudio.org/electron/jammy/amd64/rstudio-2023.03.0-386-amd64.deb
apt install -f ./rstudio-2023.03.0-386-amd64.deb
cd /

rm rstudio-2023.03.0-386-amd64.deb zotero64.tar.bz2

18) install firefox deb (no snap)

cat <<EOF > /etc/apt/preferences.d/nosnap
Package: snapd
Pin: release *
Pin-Priority: -1
EOF

cat <<EOF> /etc/apt/preferences.d/firefox-for-nosnaps
Package: firefox*
Pin: release o=Ubuntu*
Pin-Priority: -1

Package: *
Pin: release o=LP-PPA-mozillateam
Pin-Priority: 99
EOF

cat <<EOF> /etc/apt/apt.conf.d/50unattended-upgrades-firefox
Unattended-Upgrade::Allowed-Origins:: "LP-PPA-mozillateam:${distro_codename}";
EOF

add-apt-repository ppa:mozillateam/ppa
apt install firefox

19) update and upgrade software

apt-get update

apt-get upgrade

apt-get dist-upgrade

20) Remove unused packages

apt-get autoremove -y

21) Generate locales

dpkg-reconfigure locales

22) Reconfigure resolvconf

dpkg-reconfigure resolvconf

23) Configure network-manager

cat <<EOF > /etc/NetworkManager/NetworkManager.conf

[main]

rc-manager=resolvconf

plugins=ifupdown,keyfile

dns=dnsmasq

[ifupdown]

managed=false

EOF

24) Reconfigure network-manager

dpkg-reconfigure network-manager

25) Cleanup the chroot environment

truncate -s 0 /etc/machine-id

rm /sbin/initctl

dpkg-divert --rename --remove /sbin/initctl

apt-get clean

rm -rf /tmp/* ~/.bash_history

umount /proc

umount /sys

umount /dev/pts

export HISTSIZE=0

exit

sudo umount $HOME/live-ubuntu-from-scratch/chroot/dev

sudo umount $HOME/live-ubuntu-from-scratch/chroot/run

26) Create the CD image directory and populate it

cd $HOME/live-ubuntu-from-scratch

27) create 3 directory (casper, isolinux, install) in the directory image

mkdir -p image/{casper,isolinux,install}

sudo cp chroot/boot/vmlinuz-**-**-generic image/casper/vmlinuz

sudo cp chroot/boot/initrd.img-**-**-generic image/casper/initrd

cd $HOME/live-ubuntu-from-scratch

touch image/biobuntu

28) create menu of live-(DVD or USB)

cat <<EOF > image/isolinux/grub.cfg

search --set=root --file /biobuntu

insmod all_video

set default="0"

set timeout=30

menuentry "Try Biobuntu without installing" {

linux /casper/vmlinuz boot=casper quiet splash ---

initrd /casper/initrd

}

menuentry "Install Biobuntu" {

linux /casper/vmlinuz boot=casper only-ubiquity quiet splash ---

initrd /casper/initrd

}

menuentry "Check disc for defects" {

linux /casper/vmlinuz boot=casper integrity-check quiet splash ---

initrd /casper/initrd

}

EOF

29) Create manifest

cd $HOME/live-ubuntu-from-scratch

sudo chroot chroot dpkg-query -W --showformat='${Package} ${Version}\n' | sudo tee image/casper/filesystem.manifest

sudo cp -v image/casper/filesystem.manifest image/casper/filesystem.manifest-desktop

sudo sed -i '/ubiquity/d' image/casper/filesystem.manifest-desktop

sudo sed -i '/casper/d' image/casper/filesystem.manifest-desktop

sudo sed -i '/discover/d' image/casper/filesystem.manifest-desktop

sudo sed -i '/laptop-detect/d' image/casper/filesystem.manifest-desktop

sudo sed -i '/os-prober/d' image/casper/filesystem.manifest-desktop

30) Compress the chroot

cd $HOME/live-ubuntu-from-scratch

sudo mksquashfs chroot image/casper/filesystem.squashfs

printf $(sudo du -sx --block-size=1 chroot | cut -f1) > image/casper/filesystem.size

31) Create diskdefines

cd $HOME/live-ubuntu-from-scratch

cat <<EOF > image/README.diskdefines

#define DISKNAME  Biobuntu

#define TYPE  binary

#define TYPEbinary  1

#define ARCH  amd64

#define ARCHamd64  1

#define DISKNUM  1

#define DISKNUM1  1

#define TOTALNUM  0

#define TOTALNUM0  1

EOF

32) Create ISO Image for a LiveCD (BIOS + UEFI)

cd $HOME/live-ubuntu-from-scratch/image

grub-mkstandalone    --format=x86_64-efi    --output=isolinux/bootx64.efi    --locales=""    --fonts=""    "boot/grub/grub.cfg=isolinux/grub.cfg"

(
   cd isolinux &&    dd if=/dev/zero of=efiboot.img bs=1M count=10 &&    sudo mkfs.vfat efiboot.img &&    LC_CTYPE=C mmd -i efiboot.img efi efi/boot &&    LC_CTYPE=C mcopy -i efiboot.img ./bootx64.efi ::efi/boot/
)

grub-mkstandalone    --format=i386-pc    --output=isolinux/core.img    --install-modules="linux16 linux normal iso9660 biosdisk memdisk search tar ls"    --modules="linux16 linux normal iso9660 biosdisk search"    --locales=""    --fonts=""    "boot/grub/grub.cfg=isolinux/grub.cfg"

cat /usr/lib/grub/i386-pc/cdboot.img isolinux/core.img > isolinux/bios.img

33) Create iso from the image directory using the command-line

sudo xorriso    -as mkisofs    -iso-level 3    -full-iso9660-filenames    -volid "Ubuntu"    -eltorito-boot boot/grub/bios.img    -no-emul-boot    -boot-load-size 4    -boot-info-table    --eltorito-catalog boot/grub/boot.cat --grub2-boot-info  --grub2-mbr /usr/lib/grub/i386-pc/boot_hybrid.img -eltorito-alt-boot -e EFI/efiboot.img  -no-emul-boot  -append_partition 2 0xef isolinux/efiboot.img -output "../biobuntu.iso" -graft-points  "."  /boot/grub/bios.img=isolinux/bios.img  /EFI/efiboot.img=isolinux/efiboot.img
