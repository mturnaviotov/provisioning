# Provisioning

## Intro

This repository contains a basic Ansible playbook to install a simple PXE server plus CloudInit declaration to provide a Ubuntu Desktop for the end users or Ubuntu Server instance with minimal packages based on Ubuntu server minimal live image.

By default, the Ubuntu Desktop image doesn't contain a Cloud-Init client, so anyway, interactive setup will be started when you provide the desktop image in PXE boot 

### PXE Server

Server uses a small number of services:

- Dnsmasq as DHCP/TFTP/DNS server
- Apache2 as http web server
- Iptables at NAT mode
- A second network interface should be used for isolated PXE clients setup to left intact current already installed clients which also can be set up to PXE boot.

#### PXE Server requirements

- 1 CPU
- 1 GB ram
- 5 Gb for a system
- Additional space for Linux, windows, and other boot images

#### PXE Server setting

- Settings located on the top of linux-pxe-install.yml - You should provide the master interface ('external') name and target interface settings which will be configured for serve requests from PXE clients.
- Network settings are covered in templates/netplan.yaml
- CloudInit template stored in templates/cloud-init.yaml


### Ubuntu PXE Client

- Connect to to isolated network hub or switch which is connecter to second network adapter which will be serve PXE related requests.
- Set up boot from LAN in BIOS settings for hardware or virtual machine unit.
- check templates/cloud-init.yaml and update for your needs.

* Desktop setup:

- Enable desktop-related packages like ubuntu-desktop.
- Provide default username in identity section to prepare it with one default user and SSH keys 
- Disable identity block and enable the 'users: ['']' section to allow interactive user setup - login and full name and password in the first boot action.
- See disk layout setup section later

* Server setup

- Enable identity block to provide hostname (not DNS), user name, and password for access the server instance 
- Add SSH keys to the SSH section
- Disable desktop-related packages from the packages block. Also comment ```users: ['']``` from the user-data section.
- See a disk layout and update for your requirements - /home LVM partition prepared for desktop usage.


#### PXE Client Disk layout 

First, the sda3 LVM VG partition uses 15 GB, and after installation, it resized to the entire disk usage (we don't know the physical disk size)

You can tune /home partition auto extend in late operations block also if you need to install a homeless server to provide more spase to partitions like /var/lib/

After installation you can use command like provided here ```lvextend /dev/vg/home -r -l +50%FREE``` to resize other, than /home LV partitions
I decided to keep small partitions to avoid crashes when partitions /, var/log doesn't have free inodes or space

- /        - 2 Gb
- /boot    - 512 Mb
- /home    - auto-resize to 50% LVM to be able to resize /var/lib, /var, /usr or other partitions
- /var     - 2 Gb
- /var/lib - 2Gb - docker, databases, all stuff here
- /var/log - 1Gb
- /srv     - You can create a new LVM partition and mount it by yourself
