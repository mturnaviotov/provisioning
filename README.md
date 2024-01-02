# Provisioning

## Intro

This repository contains a basic ansible playbook to install a simple PXE server plus CloudInit declaration to provide
a Ubuntu Desktop for end user with minimal packages based on ubuntu server minimal

to tune this script for a server side usage you should enable identity block to provide hostname, user name and passwrd and disable ubuntu-desktop in late operations block and remove ```users: ['']``` from user-data section

## Server

Server use a small amount of servers:

- Dnsmasq as dns/dhcp/tfftp server
- Apache2 as http image server
- Iptables at nat mode
- Second network interface should be used no server isolated TFTP-bootes hosts

## Server requirements

- 1 cpu
- 1 GB ram
- 5 Gb for a system
- Additional space for linux, windows and other boot images

## Server setting

- Settings located on pxe.yml - You should provide master interface (external) name and target interface settings which will be configured for serve requests from PXE clients.
- Network settings are covered in templates/netplan.yaml

## Disk layout 

First sda3 LVM VG partition use 15gb, after installation resized to entire disk usage (we don't know physical disk size)

You can tune /home partition autoextend in late operations block also if you need to install a homeless server

After installation you ca use ```lvextend /dev/vg/home -r -l +50%FREE``` to resize LV partitions
I decided to keep small partitions to avoid /, var/log full disk craches

- /        - 2 Gb
- /boot    - 512 Mb
- /home    - auto resize to 50% lvm to be able resize /var/lib, /var, /usr or ther partitions
- /var     - 2 Gb
- /var/lib - 2Gb - docker, databases, all stuff here
- /var/log - 1Gb
- /srv     - You can create a new LVM partition and mount by yourself
