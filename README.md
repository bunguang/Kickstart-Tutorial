# KickstartTest
A tutorial about how to use kickstart as an automated installation method to install Ubuntu on several machines.

## Step 1: Install OS on One Machine
Work with Yuan Gao, use the dvd drive to install **Ubuntu server 14.04 LTS** on 1 single machine.
Then we need to set the Network Configuration to make sure that the `apt-get` function works perfectly.<br>
Of course, before that, we need to ensure that the machines should be set to "Wake up on LAN" in your BIOS configuration.

## Step 2: Preparing Work
After installing Ubuntu on the machine, we still need to do some preparing work on it.
* Install Needed Packages
```
sudo apt-get install apache2 tftpd-hpa inetutils-inetd isc-dhcp-server xinetd
```

* Mount the ISO
```
mount -o loop ubuntu-14.04-server-amd64.iso /mnt/ubuntu
```

* Create an Ubuntu Directory Hosted by Apache
```
mkdir /var/www/html/os/ubuntu14.04_02
mkdir /var/www/html/cfg
```

* Copy all of the files from your mounted ISO to your newly created Ubuntu directory, and all of the netboot files to tftpboot.
```
cp -rf /mnt/ubuntu/ /var/www/html/ubuntu/
cp -rf /mnt/install/netboot/* /tftpboot/
```

* Configuring a TFTP Server Run by xinetd. You can also run `service tftp` to check the status.
```
sudo mkdir /tftpboot
sudo chmod -R 777 /tftpboot
sudo chown -R nobody /tftpboot
sudo cp -R ubuntu/install/netboot/* /tftpboot/
```

* Configure DHCP. Edit the base configuration of your DHCP server as you wish so that it is ready to answer requests from the network it is connected to. In this sample environment, the resulting **/etc/dhcp/dhcpd.conf**  file ends up as follows, but it should be adapted to your specific environment:
```
option domain-name-servers 192.168.2.254;
default-lease-time 600;
max-lease-time 7200;
allow bootp;
allow booting;
subnet 192.168.2.0 netmask 255.255.255.0 {
        range 192.168.2.2 192.168.2.20;
        filename "pxelinux.0";
        option broadcast-address 192.168.2.255;
        option routers 192.168.2.254;
}
```

## Step 3: Edit Kickstart and Preseed File
You should create a **ks.cfg** file and a **preseed.cfg** file in the **/var/www/html** directory.<br>
In addition, You can check the [ks.cfg](https://github.com/bunguang/KickstartTest/blob/master/ks.cfg) and [preseed.cfg](https://github.com/bunguang/KickstartTest/blob/master/preseed.cfg) for reference.

## Step 4: Use Your ks.cfg
In order for your network Ubuntu install to use your kickstart file, you have to tell it where to find it. Edit **/tftpboot/pxelinux.cfg/default** and add `ks=http://<installserver>/ks.cfg` to the append line. It should then look something like this (note that the append line is one line)
```
# D-I config version 2.0
default vesamenu.c32
prompt 10
menu TITLE PXE Boot Install Menu PRC SPA
MENU BACKGROUND backgnd.png
timeout 5
label Ubuntu-14.04-Server
kernel ubuntu-installer/amd64/linux
append vga=normal initrd=ubuntu-installer/amd64/initrd.gz ks=http://192.168.2.254/cfg/ks.cfg url=http://192.168.2.254/cfg/preseed.cfg root=/dev/rd/0 --quiet
```

You should now be able to boot another pc on the lan over the network and have it install Ubuntu automagically Smile :) You can vary the tftp and http install points to have multiple versions of Ubuntu available to install on your network.
