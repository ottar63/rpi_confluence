
## Installing Confluence on Raspberry PI

Experience with installing Jira on Raspberry PI made me decide to 
setup Confluence on 2 Raspberry PI 3bB +.
One for the dabase and one for Confluence.

I decideed to use external HD on the the Confluence server since 
I exppect it will need to use swap space.

## Setup of Database server
```
$sudo raspi-config
    Change User Password
    Network Options 
        Hostname
    Localisation Option
        Change Timezone  ( if needed )
        Change keyboard  ( if needed )
    Interface Option
        Enable SSH
    Advanced Options
        Memory split   - 16  ( runninge headless and we need all the memory we can get )
```    
# Update Raspbian
```
$sudo aptt update
$sudo aptt upgrade
```
# Install Postgresql
```
$sudo apt install postgresql  
```
# Create postgresql  user and database
```
$sudo su - postgres
$createuser -d -P confluenceuser
    will prompt for password for the new db user

$createdb -E utf8 -O confluenceuser confluence
```
## Setup of  Confluence Server

Experience with Jira on Raspberry pi is that it requires a lot of memory and also  swap.
Becuase of this I have decided to run Raspbian of hard drive insted of SD card.

#Partition  hard drive
Check if any partition on new disk is mounted 
```
$df
```
if there is listed any  /dev/sdaX  partition mounted , unmount them
```
$sudo parted /dev/sda
(parted) mktable msdos
Answer  Yes on warning that existing disk label on dev/sda will be destroyed
Make boot partition ( to get the partition alligned on 8M boundary , I selected boot partition to end on 104MB )
(parted) mkpart primary fat32 0% 104M
Make swap partition 
(parted) mkpart primary liux-swap 104M 2152M
Make root partition
(parted) mkpart primary ext4 2152M 100%
```

Create boot filesystem
```
$sudo mkfs.vfat -n BOOT -F 32 /dev/sda1
```
Create root filesystem
```
$sudo mkfs.ext4 /dev/sd3
```
## Copy  Raspbian from SD to HDD
```
$sudo mkdir /mnt/hdd
$sudo mount /dev/sda3 /mnt/hdd
ssudo mkdir /mnt/dd/boot
$sudo mount /dev/sda1 /mnt/hdd/boot

$sudo rsync -ax --progress / /boot  /mnt/hdd
```

# Change UUID of root partition
Get PARTUUID  for root partition  on hdd
```
$sudo blkid /dev/sda3
```
This will give something like this :
```
/dev/sda3: UUID="904c10bb-1517-473d-97df-340557eae5a5" TYPE="ext4" PARTUUID="7bb35a07-03"
```
Here we need the PARTUUID
Edit /mnt/hdd/boot/cmdline.txt  replace value of root=PARTUUID= with value for sda3
For the example it will be root=PARTUUID=7bb35a07-03

Edit /mnt/hdd/etc/fstab
Change PARTUUID for  /boot  and /  partiriton
add entry for swap partition
Here is example of /mnt/hdd/etc/fstab for example above:
    proc            /proc           proc    defaults          0       0
    PARTUUID=7bb35a07-01  /boot           vfat    defaults          0       2
    PARTUUID=7bb35a07-03  /               ext4    defaults,noatime  0       1
    PARTUUID=7bb35a07-02  none            swap    sw          0       0

Shutdown, remove SD card and reboot.

Disable  swap-file
$sudo dphys-swapfile swapoff
Remove  swap file
$sudo dphys-swapfile uninstall
Disable creating new swapfile on reboot
$sudo systemctl disable dphys-swapfile

Use new swap partition
$sudo mkswap /dev/sda2
$sudo swapon /dev/sda2

Check that there is 2GB swap in use :
$fee
Should show :
              total        used        free      shared  buff/cache   available
Mem:         999036       34548      891524       12876       72964      902068
Swap:       1999868           0     1999868

#Install prerequisites

Install  Oracle JDK
$sudo apt install oracle-java8-jdk

Set JAVA_HOME 
Find java home 
ls -l /etc/alternative/java
lrwxrwxrwx 1 root root 53 Jun 13 11:04 /etc/alternatives/java -> /usr/lib/jvm/jdk-8-oracle-arm32-vfp-hflt/jre/bin/java
Here Java home is /usr/lib/jvm/jdk-8-oracle-arm32-vfp-hflt/jre
Edit /etc/bash.bashrc
Add the following at the end of file:
export JAVA_HOME=/usr/lib/jvm/jdk-8-oracle-arm32-vfp-hflt/jre



Create user that should run confluence
$sudo adduser confluence 

Create install directory for confluence 
$sudo mkdir /opt/confluence
Create confluence home directory
$sudo mkdir /opt/confluence-home

download atlassian-confluence-6.15.4.tar.gz

Copy  atlassian-confluence-6.15.4.tar.gz  /opt/confluence

Change owner of /opt/confluence and /opt/confluence-home
$sudo chown -R confluence /opt/confluence /opt/confluence-home

Change to user confluence 
$sudo su - confluence

Setup confluence home directory
edit /opt/confluence/atlassian-confluence-6.15.4/confluence/WEB-INF/classes/confluence-init.properties
set 
confluence.home=/opt/confluence-home


Start Confluence
$/opt/confluence/atlassian-confluence-6.15.4/bin/start-confluence.sh

Access  Confluence configuration page 
http://<ip of rpi-conflunece server>:8090

Select Trial Installation
and Next

Click on link for "Get evaluation license"

Select Confluence (Server)

Your instance is : X not installed yet  and Generate License



Confirm that you want to install license at the IP address given  ( this should be the IP address of your Raspberry PI

Aftter license key is filled in  , click Next
Then Wait and Wait
Configure Usermanagement - Select Manage user with Confluence unless you also have Jira installed

Create admin user

After Admin user is created continue with Start Further Configuration

