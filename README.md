# KickstartTest
A tutorial about how to use kickstart as an automated installation method to install Ubuntu on several machines.

## Step 1: Install OS on One Machine
Work with Yuan Gao, use the dvd drive to install **Ubuntu server 14.04 LTS** on 1 single machine.
Then we need to set the Network Configuration to make sure that the `apt-get` function works perfectly.

## Step 2: Preparing Work
After installing Ubuntu on the machine, we still need to do some preparing work on it.
* Install Needed Packages
```
sudo apt-get install apache2 tftpd-hpa inetutils-inetd isc-dhcp-server xinetd
```

* Mount the ISO
```
mount -o loop ubuntu-14.04-server-amd64.iso /mnt/ubuntu
mount -o loop rhel-server-6.4-x86_64-dvd.iso /mnt/rhel
```

* Create an Ubuntu Directory Hosted by Apache
```
mkdir /var/www/html/os/{ubuntu14.04_02,rhel6.4}
mkdir /var/www/html/cfg
```

* Copy all of the files from your mounted ISO to your newly created Ubuntu directory, and all of the netboot files to tftpboot.
```
cp -rf /mnt/ubuntu/ /var/www/html/ubuntu/
cp -rf /mnt/rhel/   /var/www/html/rhel
cp -rf /mnt/install/netboot/* /tftpboot/
cp -rf /mnt/install/netboot/* /tftpboot/
cp -rf /var/www/html/os/rhel/isolinux/vmlinuz /tftpboot/rhel64_vmlinuz
cp -rf /var/www/html/os/rhel/isolinux/initrd.img  /tftpboot/rhel64_initrd.img
```

* Configuring a TFTP Server Run by xinetd. You can also run `service tftp` to check the status.
```
sudo mkdir /tftpboot
sudo chmod -R 777 /tftpboot
sudo chown -R nobody /tftpboot
sudo cp -R ubuntu/install/netboot/* /tftpboot/
```

* Configure DHCP. Edit the base configuration of your DHCP server as you wish so that it is ready to answer requests from the network it is connected to. In this sample environment, the resulting /etc/dhcp/dhcpd.conf  file ends up as follows, but it should be adapted to your specific environment:
```
option domain-name-servers 192.168.2.254;
default-lease-time 600;
max-lease-time 7200;
allow bootp;
allow booting;
subnet 192.168.2.0 netmask 255.255.255.0 {
        range 192.168.2.2 192.168.2.10;
        filename "pxelinux.0";
        option broadcast-address 192.168.2.255;
        option routers 192.168.2.254;
}
```

## Step 3: Test Kickstart on 2 Machines
You can check the `ks.cfg` for reference.

## Step 4: Run Kickstart on Cluster
You can check the `ks.cfg` for reference.

## Step 5: Spark Environment Setup
