# OS

Raspberry Pi Noobs

# Main Use 

## Plex Server

I didn't perform any crazy setup to get Plex up and running. The complication was truly the permissions and Samba Share orchestration. I believe the permission roles are crazy right now because of this. It's probably best to refresh this server and rely only on samba share to give the user, `plex`, the ability to view media files.

Now that I'm thinking about it, I do think I set the pi to have a static internal ip address. I don't remember how this was done and I did this before thinking about saving these kind of setup processes. I'll have to find this again the next refresh and post it here.

### Start, Stop, And Restart

```bash
sudo service plexmediaserver start 

sudo service plexmediaserver stop

sudo service plexmediaserver restart
```

## Samba Share Orchestrator

Allows an external harddrive to be accessible via the local network. I'm keeping mainly movies and videos on here, but previously computer file dumps are also on here. 

Click [here](#Samba-Share) to see instruction for this.

Click [here](#ExFat-Compatability) to see instructions for allowing the pi to read my ExFat External Harddrive.

## Anki Server

Under Construction.

Click [here](#Install-Anki) to see instruction for this.

# How To

## SSH Into The Pi

I'm not sure how, but `raspberrypi.local` is a godsend and is magic that I encountered on a forum one day.

*actual*
```bash
ssh pi@raspberrypi.local
```

*template*
```bash
ssh user@internal-ip-address
```

## Samba Share

### Instructions

1. Install Samba

    ```bash
    sudo apt-get install samba
    ```

2. Create A New Permissions Group

    ```bash
    sudo groupadd sambausers
    ```

3. Create A System User That Should Have Access To The Share

    ```bash
    sudo adduser johnsmith
    ```

4. Add The Perviously Created User To The Samba Group

    ```bash
    sudo usermod -a -G samba johnsmith
    ```

5. Create A Samba Password For This User (yes - this is different than the system password)

    ```bash
    sudo smbpasswd -a johnsmith
    ```

    * If you would like to change the Samba password use the below command:
        ```bash
        sudo smbpasswd johnsmith
        ```

6. Edit The Samba Configuration File To Define The Share Specifications

    Add the below text to the end of the file. This file lives at `/etc/samba/smb.conf`
    ```
    [connor-family-share]
    comment = This is a share drive for family files
    path = /media/exfat
    read only = no
    admin users = @sambausers
    guest ok = no
    writable = yes
    ```
    The path in the above text refers to a created location. The name within the square brackets is what is seen by sambausers. I created a directory specifcally for this external harddrive (see the defined `path` location) because I needed to give Linux the ability to read exfat formatted drives. If you have not set up your pi to read exfat, and need to, please go here.

7. Restart The Samba Service

    ```bash
    sudo service smbd restart
    ```

## ExFat Compatability

I followed a [tutorial](https://pimylifeup.com/raspberry-pi-exfat/) for this and will post the highlights below.


1. Install Necessary Packages

    ```bash
    sudo apt-get install exfat-fuse
    sudo apt-get install exfat-utils
    ```

2. Create A Mount Location

    Please verify your device is location. In the below example it is `/dev/sdb1`, but this is not a catch-all. Locate this by executing the command: `fdisk -l`
    ```bash
    sudo mkdir /media/exfat
    sudo mount -t exfat /dev/sdb1/please/verify/this/location /media/exfat
    ```

3. Retain The Mount After Reboot

    Add a new line to the `fstab` file. You will first need to find the UUID of you external drive. This can be done by issuing the following command, `sudo blkid`
    ```bash
    vim /etc/fstab
    ```
    ```
    UUID=your-uuid-here-this-needs-to-be-updated /media/exfat exfat defaults,auto,umask=000,users,rw 0 0
    ```

## Install Anki

1. Navigate To The Website And Find Their Download URL

2. CD Into The Download Directory And Download

```bash
wget https://github.com/ankitects/anki/releases/download/2.1.26/anki-2.1.26-linux-amd64.tar.bz2
```

3. Build The Tar File
```bash
tar xjf ./anki-2.1.26-linux-amd64.tar.bz2
```

4. Change Directories And Make
```bash
cd anki-2.1.26-linux-amd64
sudo make install
```

5. Clean Up 

```bash
cd ..
rm anki-2.1.26-linux-amd64.tar.bz2 
```

# Configurations

## Wifi & Bluetooth

Currently the onboard wifi and bluetooth is disabled. This was done by adding the following lines to the `/boot/config.txt` file.

```bash
dtoverlay=disable-wifi
dtoverlay=disable-bt
```

## Static IP Address

### Raspberry Pi

Currently, the Raspberry Pi is assigned to a single IP address. This was achieved by adding the following lines to the `/etc/dhcpcd.conf` file.

```bash
interface eth0
static ip_address=192.168.1.2
static routers=192.168.1.1
static domain_name_servers=192.168.1.1
```

### Netgear Router

The router needs to also know to not assign this ip address dynamically. This was set by navigating to 

Advanced > Setup > LAN Setup

# Remote Desktop

VNC Server is installed and must be started before attempting to connect from a client, such as VNC Viewer. The command to start the server is as followed:
```bash
vncserver :1
```
There is a separate password for connecting into the VNC Server. Use your dynamic password.