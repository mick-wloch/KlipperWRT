##EXTFS
opkg update && opkg install block-mount kmod-fs-ext4 kmod-usb-storage kmod-usb-ohci kmod-usb-uhci e2fsprogs fdisk

DEVICE="$(sed -n -e "/\s\/overlay\s.*$/s///p" /etc/mtab)"
uci -q delete fstab.rwm
uci set fstab.rwm="mount"
uci set fstab.rwm.device="${DEVICE}"
uci set fstab.rwm.target="/rwm"
uci commit fstab

mkfs.ext4 /dev/sda1

DEVICE="/dev/sda1"
eval $(block info "${DEVICE}" | grep -o -e "UUID=\S*")
uci -q delete fstab.overlay
uci set fstab.overlay="mount"
uci set fstab.overlay.uuid="${UUID}"
uci set fstab.overlay.target="/overlay"
uci commit fstab
mount /dev/sda1 /mnt
cp -f -a /overlay/. /mnt
umount /mnt
reboot


##SWAP
opkg update && opkg install swap-utils

dd if=/dev/zero of=/overlay/swap.page bs=1M count=512
mkswap /overlay/swap.page 
swapon /overlay/swap.page
mount -o remount,size=256M /tmp 

copy rc.local

#GIT AND GCC
opkg update && opkg install git-http gcc unzip htop

##FLUIDD AND MOONRAKER

#USING DISTFEEDS.CONF V19 - PYTHON 2
opkg update && opkg install python python-pip python-cffi python-dev

cd ~
git clone https://github.com/ihrapsa/pyserial
cd pyserial
python setup.py install
cd ~
rm -rf /root/pyserial

pip install greenlet==0.4.15 jinja2 python-can==3.3.4  

#USING DISTFEEDS.CONF V20

opkg update && opkg install python3 python3-pip python3-pyserial python3-pillow python3-tornado python3-distro libsodium --force-overwrite 

cd ~
git clone https://github.com/pypa/setuptools.git
cd setuptools
python3 setup.py install
cd ~
rm -rf /root/setuptools

pip3 install inotify-simple python-jose libnacl paho-mqtt==1.5.1

cd ~
wget https://github.com/ihrapsa/KlipperWrt/raw/main/packages/python3-lmdb%2Bstreaming-form-data_packages_1.0-1_mipsel_24kc.ipk
opkg install python3-lmdb%2Bstreaming-form-data_packages_1.0-1_mipsel_24kc.ipk

opkg install nginx-ssl

##EXTRA PACKAGES

  opkg install python3-greenlet python3-cffi python3-jinja2 kmod-usb-serial-ch341
  pip3 install dbus_next
  

#KLIPPER

git clone https://github.com/KevinOConnor/klipper.git

wget -q -O /etc/init.d/klipper https://raw.githubusercontent.com/ihrapsa/KlipperWrt/main/Services/klipper
chmod 755 /etc/init.d/klipper

/etc/init.d/klipper enable

#MOONREAKER + FLUIDD

cd ~
git clone https://github.com/Arksine/moonraker.git

mkdir ~/fluidd
wget -q -O /root/fluidd/fluidd.zip https://github.com/cadriel/fluidd/releases/latest/download/fluidd.zip && unzip /root/fluidd/fluidd.zip -d /root/fluidd/ && rm /root/fluidd/fluidd.zip
wget -q -O /root/klipper_config/moonraker.conf https://raw.githubusercontent.com/ihrapsa/KlipperWrt/main/moonraker/fluidd_moonraker.conf 
wget -q -O /etc/nginx/conf.d/fluidd.conf https://raw.githubusercontent.com/ihrapsa/KlipperWrt/main/nginx/fluidd.conf

wget -q -O /etc/init.d/moonraker https://raw.githubusercontent.com/ihrapsa/KlipperWrt/main/Services/moonraker
chmod 755 /etc/init.d/moonraker
/etc/init.d/moonraker enable
/etc/init.d/moonraker restart 

#MORENGINX

wget -q -O /etc/nginx/conf.d/upstreams.conf https://raw.githubusercontent.com/ihrapsa/KlipperWrt/main/nginx/upstreams.conf
wget -q -O /etc/nginx/conf.d/common_vars.conf https://raw.githubusercontent.com/ihrapsa/KlipperWrt/main/nginx/common_vars.conf


