#dsik utilities

## mount

macOS:

```
mount | fgrep " /Volumes"
/dev/disk2s1 on /Volumes/UNTITLED (msdos, local, nodev, nosuid, noowners)
```

Linux:

```
mount | fgrep /media
/dev/sdb2 on /media/green/rootfs type ext4 (rw,relatime)
```

## df

macOS:

```
df -H | fgrep " /Volumes"
/dev/disk2s1      32G   1.9M    32G     1%       0           0  100%   /Volumes/UNTITLED
```

Linux:

```
df -H | fgrep " /media"
/dev/sdb2       1.6G  1.2G  294M  80% /media/green/rootfs
```

## diskutil

Only on macOS?

```
diskutil list external
/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *31.9 GB    disk2
   1:                 DOS_FAT_32 ⁨UNTITLED⁩                31.9 GB    disk2s1

diskutil list external -plist
Could not find disk for -plist
grey@gamera cm % diskutil -plist list external       
diskutil: did not recognize verb "-plist"; type "diskutil" for a list
grey@gamera cm % diskutil list -plist external 
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>AllDisks</key>
	<array>
		<string>disk2</string>
		<string>disk2s1</string>
	</array>
	<key>AllDisksAndPartitions</key>
	<array>
		<dict>
			<key>Content</key>
			<string>FDisk_partition_scheme</string>
			<key>DeviceIdentifier</key>
			<string>disk2</string>
			<key>OSInternal</key>
			<false/>
			<key>Partitions</key>
			<array>
				<dict>
					<key>Content</key>
					<string>DOS_FAT_32</string>
					<key>DeviceIdentifier</key>
					<string>disk2s1</string>
					<key>MountPoint</key>
					<string>/Volumes/UNTITLED</string>
					<key>Size</key>
					<integer>31913410560</integer>
					<key>VolumeName</key>
					<string>UNTITLED</string>
					<key>VolumeUUID</key>
					<string>38F4AF4D-37EF-33DD-AA53-0FFA97FCC8C3</string>
				</dict>
			</array>
			<key>Size</key>
			<integer>31914983424</integer>
		</dict>
	</array>
	<key>VolumesFromDisks</key>
	<array>
		<string>UNTITLED</string>
	</array>
	<key>WholeDisks</key>
	<array>
		<string>disk2</string>
	</array>
</dict>
</plist>
```

```
diskutil info disk2s1
   Device Identifier:         disk2s1
   Device Node:               /dev/disk2s1
   Whole:                     No
   Part of Whole:             disk2

   Volume Name:               UNTITLED
   Mounted:                   Yes
   Mount Point:               /Volumes/UNTITLED

   Partition Type:            DOS_FAT_32
   File System Personality:   MS-DOS FAT32
   Type (Bundle):             msdos
   Name (User Visible):       MS-DOS (FAT32)

   OS Can Be Installed:       No
   Media Type:                Generic
   Protocol:                  USB
   SMART Status:              Not Supported
   Volume UUID:               38F4AF4D-37EF-33DD-AA53-0FFA97FCC8C3
   Partition Offset:          1048576 Bytes (2048 512-Byte-Device-Blocks)

   Disk Size:                 31.9 GB (31913410560 Bytes) (exactly 62330880 512-Byte-Units)
   Device Block Size:         512 Bytes

   Volume Total Space:        31.9 GB (31897812992 Bytes) (exactly 62300416 512-Byte-Units)
   Volume Used Space:         1.9 MB (1900544 Bytes) (exactly 3712 512-Byte-Units) (0.0%)
   Volume Free Space:         31.9 GB (31895912448 Bytes) (exactly 62296704 512-Byte-Units) (100.0%)
   Allocation Block Size:     16384 Bytes

   Media OS Use Only:         No
   Media Read-Only:           No
   Volume Read-Only:          No

   Device Location:           External
   Removable Media:           Removable
   Media Removal:             Software-Activated

   Solid State:               Info not available
```

### How can I see the rootfs in macOS?

Carfule, this method does not allow you to write to it.

```bash
mac $ brew cask install osxfuse
mac $ brew install ext4fuse
mac $ diskutil list external

diskutil list external                  
/dev/disk2 (external, physical):
  #:            TYPE NAME          SIZE    IDENTIFIER
  0:   FDisk_partition_scheme            *64.1 GB  disk2
  1:       Windows_FAT_32 ⁨boot⁩          268.4 MB  disk2s1
  2:           Linux ⁨⁩            1.6 GB   disk2s2
          (free space)             62.2 GB  - (edited) 

mac $ sudo ext4fuse /dev/disk2s2 /Volumes/rootfs -o allow_other
```

This will ask you to allow the fuse program to get access to your system, 
which you have to allow to use this method. Than you have to reboot the machine.
After the reboot you have to issue the command again

```bash
mac $ sudo ext4fuse /dev/disk2s2 /Volumes/rootfs -o allow_other
```

Now you can look at the files with

```bash
ls /Volumes/rootfs
```


## See Also

Some older documentation, much of it is still relevant, is available
at the following links. We will in time integrate here or update them.

TODO: does thsi work: **NOTE:** We also have additional information just started on how to
install a kubernetes cluster. This however doe not yet work. See
[kubernetes cluster instalation guide.](https://github.com/cloudmesh/cloudmesh-pi-cluster/tree/main/cloudmesh/pi/cluster/k3)
for k3s program documentation.

**NOTE**: [Old manual documentation](https://cloudmesh.github.io/cloudmesh-manual/projects/project-pi-burn.html?highlight=burn)





## STUFF TO BE DELETED OR INTEGRATED IN REST OF DOCUMENT


In this example we will set up a network of Raspberry Pi's. In our
example we will set up a cluster with 3 Pi's. We have the following
assumptions:

* We assume you have at at least 3 Raspberry Pi's. This will provide
  you in our example with a master and two worker Pi's. However if you
  have more, you can use more by adding more worker Pi's. Please adapt
  our example accordingly. The minimum number of workers is 1.

* We assume you will use the hostname for the master to be `red`, and
  for the workers you will use `red[001-002]`.

* We assume that you have one more SD Card than you have PI's. If not
  you need to reinstall the burn program on that master. and use that
  reburned SD card.

* We assume that you have an SD card writer. We wecommend that you get
  one that is USB 3 compatible as the new PI's do support USB 3 and
  thus are faster.

* We assume you have the master configured with the (latest Raspberry
  Pi
  OS)[https://www.raspberrypi.org/documentation/installation/installing-images/README.md]

* We assume you use the username `pi` on `red`. Note, that `pi` is
  the default username on Rasperry Pi OS. To rename the hostname which
  is originally `raspberry` please log into it and execute the
  following command:

  TODO: MAYBE WE SHOULD JUST LEAVE THE NAME raspberry and just burn a
  new SD card

  ```
  pi@raspberry:$ sudo echo "127.0.1.1 red" >> /etc/hosts
  pi@raspberry:$ sudo reboot
  ```

  after the reboot the machien will show up as `red`
  
In the first step we will install some useful software on the master
that allows us to easily burn and reburn also the card for master. 
On `red`, perform the following to create an ssh-key, download
cloudmesh for Pi, and download the latest raspbian OS with `cms burn`

```
pi@red:$ ssh-keygen
pi@red:$ curl -Ls http://cloudmesh.github.io/get/pi | sh
pi@red:$ source ~/ENV3/bin/activate

(ENV3) pi@red:$ ssh-add
(ENV3) pi@red:$ cms burn image get latest
(ENV3) pi@red:$ cms burn image ls
```

To prepare for burning check if your SD card writer is detected and
observe the output (note it is a prompted command). We have multiple
programs to do so likely the `info` command will be sufficient.

```
(ENV3) pi@red:$ cms burn detect
(ENV3) pi@red:$ cms burn info

...

# ----------------------------------------------------------------------
# SD Cards Found
# ----------------------------------------------------------------------

+----------+----------------------+----------+-----------+-------+------------------+---------+-----------+-----------+
| Path     | Info                 | Readable | Formatted | Empty | Size             | Aaccess | Removable | Writeable |
+----------+----------------------+----------+-----------+-------+------------------+---------+-----------+-----------+
| /dev/sdx | Generic Mass-Storage | True     | True      | False | 31.9 GB/29.7 GiB | True    | True      | True      |
+----------+----------------------+----------+-----------+-------+------------------+---------+-----------+-----------+
```

Now set your default SD card device with the following command (your
`/dev/sdx` may be different as reported by the `info` command

```
(ENV3) pi@red:$ export DEV=/dev/sdx
```

and start burning `red[001-002]`.

```
(ENV3) pi@red:$ cms burn create \
--hostname=red[001-002]
```

After you put the SD Cards in the worker Pis and boot them you can log
into them with

```
(ENV3) pi@red:$ ssh pi@red001.local
```

In the future, we will try to remove the `pi` user. 

E.g. use we will integrate `cms host ssh config red[001-003]`


Note that if the Pi's are all connected under (cms
bridge)[https://github.com/cloudmesh/cloudmesh-pi-cluster/tree/main/cloudmesh/bridge],
the `.local` extension is not necessary.


>
> ***Alternative: Specifying --device***
>
> If you would rather not use `export DEV=/dev/sdx`, you can specify
> it using the `--device` option:
>
> ```
> (ENV3) pi@red:$ cms burn create \
> --device=/dev/sdx \
> --hostname=red[001-002] \
> --ipaddr=169.254.10.[1-2]
> ```
>

>
> ***Alternative: Using WiFi:***
>
> If you want to connect your workers directly to the internet via
> WiFi, then you only need to add the following two lines to the end
> of `cms burn`:
>
> ```
> (ENV3) pi@red:$ cms burn create \
> --hostname=red[001-002] \
> --ipaddr=192.168.1.10 \
> --ssid=MyWifiRouterName \
> --wifipassword=MyWifiPassword
> ```
>

>
> ***Alternative: Burning with Multiple SD Card Writers:***
>
> To use multiple card writers you can use
>
> ```
> (ENV3) pi@red:$ cms burn create \
> --device=/dev/sd[a,e] \
> --hostname=red[001-002] \
> --ipaddr=169.254.10.[1-2]
> ```
>



THAT GREGOR DID NOT WANT TO DELETE  AS IT COULD BE USEFUL
AND MAYBE COULD BE INTEGRATED IN THE MAIN DOCUMENTATION

**NOTE**: Notice I have changed the `--ipaddr` option. This is to
remind everyone that the static IP must fall into your network
range. Many home networks have a 192.168.1.x network range, which is
why I have set up the example in this context.

## Step 4(alt). Burning Multiple Cards

The process for burning multiple cards is very straightforward and
analogous to burning a single card. In this example, we assume we want
hostnames `red001, red002, red003` with ip addresses `169.254.10.1,
169.254.10.2, 169.254.10.3' burned on cards located at `/dev/sda,
/dev/sde, /dev/sdf` respectively. Our command is as follows:

```
(ENV3) pi@red:$ cms burn create \
--image=latest \
--device=/dev/sd[a,e,f] \
--hostname=red[001-003] \
--sshkey=default \
--ipaddr=169.254.10.[1-3]
```

This has not yet been tested due to lack of card-readers.

## DEPRECATED. DO NOT GO BEYOND THIS LINE AS THE DOCUMENTATION IS OUT OF DATE


### Installation

First, you must install `cms burn`. In a future version, this will be done
with

```bash
$ pip install cloudmesh-pi-burn
```
   
However, in the meanwhile, you do it as follows:

```bash
$ mkdir cm
$ cd cm $ git clone https://github.com/cloudmesh/cloudmesh-pi-burn.git
$ cd cloudmesh-pi-burn
$ pip install -e .
```    

In the future, we will remove the -e

```bash
$ pip install .
```

### Information about the SD Cards and Card Writer

You need at least one SD Card writer. However, `cms burn` is
supposed to work also with a USB hub in which you can plug in
multiple SD Cards and burn one card at a time. Once done, you can add a
new batch, and you can continue writing. This is done for all specified
hosts so that you can minimize the interaction with the SD cards.

To find out more about the Card writers and the SD Cards, you can use
the command

```bash
$ cms burn detect
```

It will first ask you to not plug in the SDCard writer to probe the
system in empty status. Then you need to plug in the SD Card writer
and with the cards in it. After you have said yes once you plugged
them in, you will see an output similar to: 

```
# ----------------------------------------------------------------------
# Detecting USB Card Reader
# ----------------------------------------------------------------------

Make sure the USB Reader is removed ...
Is the reader removed? (Y/n) 
Now plug in the Reader ...
Is the reader pluged in? (Y/n) 

# ----------------------------------------------------------------------
# Detected Card Writer
# ----------------------------------------------------------------------

Bus 002 Device 020: ID 045b:0210 Hitachi, Ltd 
Bus 002 Device 024: ID 05e3:0749 Genesys Logic, Inc. 
Bus 001 Device 014: ID 045b:0209 Hitachi, Ltd 
Bus 001 Device 015: ID 045b:0209 Hitachi, Ltd 
Bus 001 Device 016: ID 05e3:0749 Genesys Logic, Inc. 
Bus 002 Device 023: ID 05e3:0749 Genesys Logic, Inc. 
Bus 002 Device 019: ID 045b:0210 Hitachi, Ltd 
Bus 002 Device 021: ID 05e3:0749 Genesys Logic, Inc. 
Bus 002 Device 022: ID 05e3:0749 Genesys Logic, Inc. 
```

Note that in this case, we will see two devices, one for the USB hub in
which the card is plugged in, and one for the SD Card itself.

Next, we like to show you a bit more useful information while probing
the operating system when the SD Card  Writers are plugged in. Please
call the command:

```bash
$ cms burn info
```

You will see an output similar to 

```
# ----------------------------------------------------------------------
# Operating System
# ----------------------------------------------------------------------

Disk /dev/mmcblk0: 29.7 GiB, 31914983424 bytes, 62333952 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x5e3da3da

Device         Boot  Start      End  Sectors  Size Id Type
/dev/mmcblk0p1        8192   532479   524288  256M  c W95 FAT32 (LBA)
/dev/mmcblk0p2      532480 62333951 61801472 29.5G 83 Linux

# ----------------------------------------------------------------------
# SD Cards Found
# ----------------------------------------------------------------------

+------+----------+--------+----------+-------+-------------------+-----------+-----------+
| Name | Device   | Reader | Formated | Empty | Size              | Removable | Protected |
+------+----------+--------+----------+-------+-------------------+-----------+-----------+
| sde  | /dev/sde | True   | True     | True  |  31.9 GB/29.7 GiB | True      | False     |
| sdd  | /dev/sdd | True   | True     | False |  31.9 GB/29.7 GiB | True      | False     |
| sdc  | /dev/sdc | True   | True     | True  |  31.9 GB/29.7 GiB | True      | False     |
| sdb  | /dev/sdb | True   | True     | False |  31.9 GB/29.7 GiB | True      | False     |
| sda  | /dev/sda | True   | True     | False |  31.9 GB/29.7 GiB | True      | False     |
+------+----------+--------+----------+-------+-------------------+-----------+-----------+
```

Under `Operating System` you will see the block device you will see
information about your operating system. This is the card plugged into
the back of your PI.

Under SDCards found you will see the list of SD Cards and some
information about the cards that are plugged into the writers.

Make sure that you only include cards that you truly want to overwrite.
We have given an example where this is not the case while indicating it
in the Empty column. We recommend that you only use formatted cards, so
you are sure you do not by accident delete information.


### Finding Image Versions

Start using sudo now!

First, you have to find the raspbian image you like to install. For this
purpose, we have developed a command that lists you the available images
in the Raspberry Pi repository. To see the versions, please use the
command

```bash
# cms burn image versions
```

Once in a while, they come out with new versions. You can refresh the
list with

```bash
# cms burn image versions --refresh
```

### Downloading an Image

To download the newest image, use the command

```bash
# cms burn image get latest
```

The image is downloaded into the folder

* `~/.cloudmesh/cmburn/images`

To list the downloaded images, you can use the command

```bash
# cms burn image ls
```

In case you like to use the latest download, you can use the
command.

TODO: MISSING

You can also specify the exact URL with

```bash
# cms burn image get https://downloads.raspberrypi.org/raspbian_lite/images/raspbian_lite-2019-09-30/2019-09-26-raspbian-buster-lite.zip
```

### ROOT

For the burn process, you need to use root privileges. To achieve this,
you need to execute the following commands. The source command
activates the python virtual env that you have created where you
installed the `cms burn` command

```bash
$ sudo su
# source /home/pi/ENV3/bin/activate
```

Please note that for our notation a `#` indicates this command is
executed in root.

### Creating Cluster SD-Cards

Next, we describe how we create a number of SD-Cards to create a cluster.
Each card will have a unique hostname, an IP address and your public key. 
To locate your device, you can use:

```bash
$ cms burn info  
```

You can look at the names of your devices under the device column. Eg
/dev/sda,/dev/sdb,etc

### Burning SD-Cards

To burn one card, we will use `cms burn create` with several
important options:

* `--image` specifies the name of the image to burn
* `--device` is the path to the SD card. If this option is omitted,
  then `cms burn` will use the devices listed under `cms burn info`
* `--hostname` is the name for the pi
* `--sshkey` is the path to your SSH PUBLIC key
* `--blocksize` specified to 4M for our purposes

If you want to specify a password for desktop login (for debugging
purposes), you can use the option

* `--passwd=PASSWD` 

to set a password. In the future, you should not use this option as we
do not want to login through the terminal. We only want to SSH from the
master Pi.

### Auto Format to FAT32

The `--format` is an option that can be used to automatically format
your SD card to FAT32. The current implementation is quite unstable as
it makes several assumptions. If this option is used, it will most
likely work.

Do not worry if you see the message `No partition is defined yet!`

### Note on using a static IP address

You can use the `--ipaddr=IP` option to set a static IP address for your
Pis. To make sure this works, ensure that your master Pi is connected to
the network as cms burn will pull information from the network to
configure static IP usage.

For more information on options, see `/cmburn/pi/cmpiburn.py`

Here is an example call of the command `create` using a static IP
address connecting to a home wifi network

TODO: WIFI is not listed here

```bash
# cms burn create \
    --image=2020-02-05-raspbian-buster-lite \
    --device=/dev/sda \
    --hostname=red2 \
    --sshkey=/home/pi/.ssh/id_rsa.pub \
    --blocksize=4M \
    --ipaddr=169.254.10.30 \
    --format
```

Here we are assuming that your device name is sda, but its very important
to verify it once before executing the above command. Note that if we
omit the `--device` option, then `cms burn` will refer to the devices
listed using `cms burn info`

If your Pis are going to use ethernet connection, then the command is as
simple as:

```bash
# cms burn create \
    --image=2020-02-05-raspbian-buster-lite \
    --device=/dev/sda \
    --hostname=red2 \
    --sshkey=/home/pi/.ssh/id_rsa.pub \
    --blocksize=4M \
    --ipaddr=169.254.10.32 \
    --format
```

To burn many cards, you can specify them conveniently in parameter
notation in  the `--hostname` and `--ipaddr` arguments:

```bash
# cms burn create \
    --image=2020-02-05-raspbian-buster-lite \
    --device=/dev/sd[a-f]
    --hostname=red[2-7] \
    --sshkey=/home/pi/.ssh/id_rsa.pub \
    --blocksize=4M \
    --ipaddr=169.254.10.[32-37] \
    --format
```

Note the ranges are inclusive. Alternatively, we can omit the --device
option and allow cms burn to detect the devices from `cms burn
info`:

```bash
# cms burn create \
    --image=2020-02-05-raspbian-buster-lite \
    --hostname=red[2-7] \
    --sshkey=/home/pi/.ssh/id_rsa.pub \
    --blocksize=4M \
    --ipaddr=169.254.10.[32-37] \
    --format
```


You may see the program output some unmount errors during the burn
process - this is normal.

After the process is completed, a message will appear on your terminal
stating the number of cards you have burnt.

You can verify if the burn process is completed or not by plugging in
one of the SD cards to a Raspberry Pi and starting it. Raspberry Pi
terminal appears asking your login and password. After the successful
authentication, now you can use your raspberry pi just like any other.


Here is an alternative version to the command above with a different
`--device` option.


```bash
# cms burn create \
    --image=2020-02-05-raspbian-buster-lite \
    --device=/dev/sda \
    --hostname=red[2-7] \
    --sshkey=/home/pi/.ssh/id_rsa.pub \
    --blocksize=4M \
    --ipaddr=169.254.10.[32-37] \
    --format
```

Notice here how we have only listed one port in the `--device` option.
This would be in the case that we only have one SD card writer, but we
don't want to rerun the command each time. That would be quite tedious.
Instead, the command will burn to `/dev/sda` with hostname red2, then a
prompt will come up asking the user if we want to reuse `/dev/sda`.

```
Slot /dev/sda needs to be reused. Do you wish to continue? [y/n] 
# y
Insert next card and press enter...
# [enter]
Burning next card...
```

In this way, we avoid having to rerun the command while providing enough
safeguards so we don't accidentally overwrite the last SD card. This
prompt will also appear if the number of hosts (in this example there
are 4 hosts) exceeds the number of available devices (1 in this
example).

If the only device listed under `cms burn info` is `/dev/sda`, then the
above command is equivalent to:

```bash
# cms burn create \
    --image=2020-02-05-raspbian-buster-lite \
    --hostname=red[2-7] \
    --sshkey=/home/pi/.ssh/id_rsa.pub \
    --blocksize=4M \
    --ipaddr=169.254.10.[32-37] \
    --format
```


### From the raspberry FAQ

Quote:
    There is no on/off switch! To switch on, just plug it in. To switch
    off, if you are in the graphical environment, you can either log out
    from the main menu, exit to the Bash prompt, or open the terminal.
    From the Bash prompt or terminal, you can shut down the Raspberry Pi
    by entering sudo halt `-h`. Wait until all the LEDs except the power
    LED are off, then wait an additional second to make sure the SD card
    can finish its wear-leveling tasks and write actions. You can now
    safely unplug the Raspberry Pi. Failure to shut the Raspberry Pi
    down properly may corrupt your SD card, which would mean you would
    have to re-image it.
    


## Alternative Network Setup

As implemented, cms bridge creates two seperate networks with the
master pi acting as a DHCP server and router for the works. Workers
traffic on the physical LAN is forwarded via the masters WLAN0 to the
internet when required.  The WLAN and LAN are two seperate networks.

An alternative option is to follow option 1 here
<https://willhaley.com/blog/raspberry-pi-wifi-ethernet-bridge/>.  This
setup uses Proxy ARP and IP forwarding to create a single, flat
network using the existing DHCP server (for instance from a home
router). The WLAN and LAN all have ip addresses on the same
subnet. LAN traffic is still forwarded via the masters WLAN interface
when required. An advantage of this setup is its simplicty. There is
no need for ssh tunnelling to access workers via a lapton on the
wireless network, you can simply youse local mDNS resolotion (i.e. ssh
pi@red001.local from a laptop on the WLAN). This option is not a good
choice if the cluster does not have

I also think it would be easier to extend cms bridge to provide
firewall capabilities. Since it essentially already using linux
fireall utils (iptables) to provide the forwarding.

When we scale to the 10s of nodes will have to have the cluster
plugged directly into a network with wired access to DHCP and
internet. That might exceed a standard home routers capabilities. We
will might be looking for a managed switch at that point, where we can
setup DHCP directly on the switch.
