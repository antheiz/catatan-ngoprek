# Menginstall TP-Link Archer T4U Driver pada Linux Ubuntu
---
Langkah-langkah :
- Pastikan USB TP-Link tidak Terpasang
- Buka Terminal
- Ketik Perintah:
    - ```git clone https://github.com/cilynx/rtl88x2bu.git```
    - ```cd rtl88x2bu```
    - ```VER=$(sed -n 's/\PACKAGE_VERSION="\(.*\)"/\1/p' dkms.conf)```
    - ```sudo rsync -rvhP ./ /usr/src/rtl88x2bu-${VER}```
    - ```sudo dkms add -m rtl88x2bu -v ${VER}```
    - ```sudo dkms build -m rtl88x2bu -v ${VER}```
    - ```sudo dkms install -m rtl88x2bu -v ${VER}```
    - ```sudo modprobe 88x2bu```
- Pasangkan USB TP-Link pada Komputer
- TP-Link Archer T4U Driver pada Linux Ubuntu, Berhasil di install.
