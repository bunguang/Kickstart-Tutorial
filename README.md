# KickstartTest
A tutorial about how to use kickstart as an automated installation method to install Ubuntu on several machines.

## Step 1: Install OS on One Machine
Use the dvd drive to install **Ubuntu server 14.04 LTS** on 1 single machine.
Then we need to make sure that the `apt-get` function works perfectly, for we need to download some packages from the Internet.<br>
Of course, we still need to do some configuration with the **/etc/network/interfaces**, to set the IP address of the DHCP server in the LAN. Add this to the end of the file (You should replace `em0` with your own network interface connected to the LAN.)
```
auto em0
iface em0 inet static
address 192.168.2.254
netmask 255.255.255.0
```

## Step 2: Preparing Work
After installing Ubuntu on the machine, we still need to do some preparing work on it.
* Install Needed Packages
```
sudo apt-get install apache2 tftpd-hpa isc-dhcp-server xinetd
```

* Mount the ISO
```
mount -o loop ubuntu-14.04-server-amd64.iso /mnt/ubuntu
```

* Create an Ubuntu Directory Hosted by Apache
```
mkdir /var/www/html/ubuntu
mkdir /var/www/html/cfg
```

* Copy all of the files from your mounted ISO to your newly created Ubuntu directory.
```
cp -rf /mnt/ubuntu/* /var/www/html/ubuntu/
```

* Configuring a TFTP Server Run by xinetd. Edit **/etc/default/tftpd-hpa**, and the resulting file should look like this
```
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/tftpboot"
TFTP_ADDRESS="0.0.0.0:69"
TFTP_OPTIONS="-l -c -s"
```

* Create new tftpboot directory and copy all of the netboot files to the tftpboot.
```
sudo mkdir /tftpboot
sudo chmod -R 777 /tftpboot
sudo chown -R nobody /tftpboot
sudo cp -R /var/www/html/ubuntu/install/netboot/* /tftpboot/
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

* Restart all the services you need. You can also use `service <servcieName> status` to check the status.
```
/etc/init.d/xinetd start
/etc/init.d/isc-dhcp-server restart
/etc/init.d/tftpd-hpa restart
```

## Step 3: Edit Kickstart and Preseed File
* You should create a **ks.cfg** file and a **preseed.cfg** file under the **/var/www/html** directory. In addition, You can check the [ks.cfg](https://github.com/bunguang/KickstartTest/blob/master/ks.cfg) and [preseed.cfg](https://github.com/bunguang/KickstartTest/blob/master/preseed.cfg) for reference.

* In order for your network Ubuntu install to use your kickstart file, you have to tell it where to find it. Edit **/tftpboot/pxelinux.cfg/default**, and it should then look something like this
```
# D-I config version 2.0
default vesamenu.c32
prompt 10
menu TITLE PXE Boot Install Menu PRC SPA
MENU BACKGROUND backgnd.png
timeout 5
label Ubuntu-14.04-Server
kernel ubuntu-installer/amd64/linux
append vga=normal initrd=ubuntu-installer/amd64/initrd.gz ks=http://192.168.2.254/ks.cfg url=http://192.168.2.254/preseed.cfg root=/dev/rd/0 --quiet
```

## Step 4: Autoboot
You should now be able to boot another pc on the LAN over the network and have it install Ubuntu automagically. All you need to do is to restart other machines on the LAN and choose **Network Booting** in their BIOS interfaces.
