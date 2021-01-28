# Cloudmesh Pi Burner for SD Cards

**WARNING:** *This program is designed for a Raspberry Pi and must not
be executed on your laptop or desktop. An earlier version that could
be run on **Linux, macOS, and Windows 10 is no longer supported**. If
you want to help us porting them on any of these OSes, please contact
laszewski@gmail.com*


[![image](https://img.shields.io/travis/TankerHQ/cloudmesh-pi-burn.svg?branch=main)](https://travis-ci.org/TankerHQ/cloudmesn-pi-burn)
[![image](https://img.shields.io/pypi/pyversions/cloudmesh-pi-burn.svg)](https://pypi.org/project/cloudmesh-pi-burn)
[![image](https://img.shields.io/pypi/v/cloudmesh-pi-burn.svg)](https://pypi.org/project/cloudmesh-pi-burn/)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)


<!--TOC-->

- [Cloudmesh Pi Burner for SD Cards](#cloudmesh-pi-burner-for-sd-cards)
  - [Introduction](#introduction)
  - [Nomenclature](#nomenclature)
  - [Quickstart for Bridged WiFi](#quickstart-for-bridged-wifi)
    - [Requirements](#requirements)
    - [Master Pi](#master-pi)
    - [Single Card Burning](#single-card-burning)
    - [Burning Multiple SD Cards with a Single Burner](#burning-multiple-sd-cards-with-a-single-burner)
    - [Connecting Pis to the Internet via Bridge (OPTION 1)](#connecting-pis-to-the-internet-via-bridge-option-1)
  - [Quickstart Guide for Mesh Networks](#quickstart-guide-for-mesh-networks)
  - [Set up of the SSH keys and SSH tunnel](#set-up-of-the-ssh-keys-and-ssh-tunnel)
  - [Manual Pages](#manual-pages)
    - [Manual Page for the `burn` command](#manual-page-for-the-burn-command)
    - [Manual Page for the `bridge` command](#manual-page-for-the-bridge-command)
    - [Manual Page for the `host` command](#manual-page-for-the-host-command)
  - [FAQ/Hints](#faqhints)

<!--TOC-->

## Introduction

`cms burn` is a program to burn many SD cards for the preparation of
building clusters with Raspberry Pi's. It allows users to create
readily bootable SD cards that have the network configured, contain a
public ssh key from your machine that you used to configure the
cards. Thus not much additional setup is needed for a cluster. Another
unique feature is that you can burn multiple cards in a row, each with
their individual setup such as hostnames and ipadresses.


## Nomenclature

* Commands proceeded with `pi@red:$` are to be executed on the
  Rasperry Pi with the name red.

* Commands with `(ENV3) pi@red:$` are to be executed in a virtula ENV
  using Python 3 on the Raspberry Pi with the name red
  
## Quickstart for Bridged WiFi

To provide you with a glimpse on what you can do with cms burn, we
have provided this quickstart guide that will create one master PI and
several workers.

This setup is intended for those who have restricted access to their
home network (ie. cannot access router controls).  For example, those
on campus WiFis or regulated apartment WiFis.

The Figure 1 describes our network configuration. We have 5
Raspberry Pi 4s: 1 master and 4 workers. We have WiFi access, but we
do not necessarily have access to the router's controls.

We also have a network switch, where the master and workers can
communicate locally, but we will also configure the master to provide
internet access to devices on the network switch via a "network
bridge".

![](https://github.com/cloudmesh/cloudmesh-pi-burn/raw/main/images/network-bridge.png)

Figure 1: Pi Cluster setup with bridge network

### Requirements

For the quickstart we have the following requirements:

* SD Cards and Raspberry Pis
  
* Master Pi: You will need at least **1 Raspberry Pi** SD Card burned
  using [Raspberry Pi imager](https://www.raspberrypi.org/software/).
  You can use your normal operating system to burn such a card
  including Windows, macOS, or Linux.  Setting up a Raspberry Pi in
  this manner should be relatively straightforward as it is nicely
  documented online (For example,
  [how to setup SSH](https://www.raspberrypi.org/documentation/remote-access/ssh/)).
  All you will need for this guide is an internet connection for your
  Pi. It might also be of use to change the hostname of this Pi.

* You will need an SD card writer (USB tends to work best) to burn new
  cards We recommend that you invest in a USB3 SDCard writer as they
  are significantly faster and you can resuse them on PI'4s

### Master Pi

First we need to configure the Master Pi

**Step 1.** Installing Cloudmesh on the Master Pi

Update pip and the simple curl command below will generate an ssh-key, update your
system, and install cloudmesh.

```
pi@masterpi:~ $ pip install pip -U
pi@masterpi:~ $ curl -Ls http://cloudmesh.github.io/get/pi | sh
```

This will take a moment...

**Step 2.** Activate Python Virtual Environment

Activate Python Virtual Environment, 
if you have not already, enter the Python virtual environment provided
by the installation script.

```
pi@masterpi:~ $ source ~/ENV3/bin/activate
```

There is currently an issue with the version of numpy the pi4 has installed. To fix run the below command. To see more info about see <https://numpy.org/devdocs/user/troubleshooting-importerror.html#raspberry-pi>.

```
(ENV3) pi@masterpi:~ $ sudo apt-get install libatlas-base-dev
```

To verify run the below command. You should see no errors.

```
(ENV3) pi@masterpi:~ $ cms help
```

**Step 3.** Download the latest Raspberry Pi Lite OS

The following command will download the latest images for Raspberry
Lite OS.

```
(ENV3) pi@masterpi:~ $ cms burn image get latest
```

We can verify our image's downloaded with the following.

```
(ENV3) pi@masterpi:~ $ cms burn image ls
```

**Step 4**. Setup SD Card Writer

Run the following command to setup your SD Card Writer with cms
burn. It will provide a sequence of instructions to follow.

```
(ENV3) pi@masterpi:~ $ cms burn detect

Make sure the USB Writer(s) is removed ...
Is the writer(s) removed? y/n
Now plug in the Writer(s) ...
Is the writer(s) plugged in? y/n

# ----------------------------------------------------------------------
# Detected Card Writers
# ----------------------------------------------------------------------

Bus 001 Device 003: ID 1908:0226 GEMBIRD
```

Now insert a FAT32 formatted SD cards into your writer to create worker 
SD cards.

Running the following command will provide us information on our SD
card's location on the system.

```
(ENV3) pi@masterpi:~ $ cms burn info
...
# ----------------------------------------------------------------------
# SD Cards Found
# ----------------------------------------------------------------------

+----------+----------------------+----------+-----------+-------+------------------+---------+-----------+-----------+
| Path     | Info                 | Readable | Formatted | Empty | Size             | Aaccess | Removable | Writeable |
+----------+----------------------+----------+-----------+-------+------------------+---------+-----------+-----------+
| /dev/sda | Generic Mass-Storage | True     | True      | False | 64.1 GB/59.7 GiB | True    | True      |           |
+----------+----------------------+----------+-----------+-------+------------------+---------+-----------+-----------+
```

> `cms burn info` has other useful information, but for the purposes of this guide we omit it. 

We can see from the information displayed that our SD card's path is
`/dev/sda`. Of course, this may vary. Let us record this path for `cms
burn` access.

```
(ENV3) pi@masterpi:~ $ export DEV=/dev/sda
```

`cms burn` is now properly configured and ready to begin burning
cards. See the following sections on burning that are in accordance
with your setup.

### Single Card Burning

Step 0. Ensure the SD card is inserted.

We can run `cms burn info` again as we did above to verify our 
SD card is connected.

Step 1. Burning the SD Card

Choose a hostname for your card. We will use `red001`.

```
(ENV3) pi@masterpi:~ $ cms burn create --hostname=red001
```

Wait for the card to burn. Once the process is complete, it is safe 
to remove the SD card.


### Burning Multiple SD Cards with a Single Burner

Step 0. Ensure the first SD card is inserted into the burner.

We can run `cms burn info` again as we did above to verify our SD 
card is connected.

Step 2. Burning the Cards

`cms burn` supports logical incremenation of numbers/characters.

For example, `red00[1-2]` is interpreted by cms burn as `[red001, red002]`.
Similarly, `red[a-c]` is interpreted by cms burn as `[reda, redb, redc]`.

We can burn 2 SD cards as follows:

```
(ENV3) pi@masterpi:~ $ cms burn create --hostname=red00[1-2]
```

The user will be prompted to swap the SD cards after each card burn if 
there are still remaining cards to burn.

One if the important aspects is how to set up networking. We have
three options

OPTION 1: The framework we use to set up default networking will use a DHCP
server. This is configured at a later step with the command `cms
bridge` that manages all ip addresses on the master. This is the
easiest way to set up networking. There are two other options

OPTION 2: Setup static IPs via the bridge command. `cms bridge` allows
users to assign static IPs to nodes in their cluster. See
[cms bridge documentation](https://github.com/cloudmesh/cloudmesh-pi-cluster/blob/main/cloudmesh/bridge/README.md#a-simple-command-to-setup-a-network-bridge-between-raspberry-pis-and-a-manager-pi-utilizing-dnsmasq)
for more information.

OPTION 3: If the user wishes to strictly assign a static IP at the
time of burning, they may use the `--ipaddr=IP` as noted in the
[cms burn manual](https://github.com/cloudmesh/cloudmesh-pi-burn#manual-burn). The
behavior of this parameter is very similar to the hostnames
parameter. For example, `10.1.1.[1-3]` evaluates to
`[10.1.1.1, 10.1.1.2, 10.1.1.3]`

Which option you use may depend on your persoanl preferences or your
network requirements. If in doubt, start with OPTION 1.

### Connecting Pis to the Internet via Bridge (OPTION 1)

Figure 1 depicts how the network is set up with the help of the bridge command.

![](https://github.com/cloudmesh/cloudmesh-pi-burn/raw/main/images/network-bridge.png)

Figure 1: Networking Bridge

Step 0. Recap and Setup

At this point we assume that you have used `cms burn` to create all SD cards for the
Pi's.

We are also continuing to use `masterpi` (which is where we burn the worker SD cards).

We will now use `cms bridge` to connect the worker Pis to the
internet. Let us again reference the diagram of our network setup. You
should now begin connecting your Pis together via network
switch. Ensure that `masterpi` is also connected into the network
switch.

Step 1. Verify Local Connection to Workers

Ensure your workers are booted and that your network switch is turned
on. Once the Pis are done booting up, we will verify our local
connections to them on the network switch via SSH.


> Note: To figure out when a Pi is done completing its initial bootup process, 
> the green light on the Pi will flash periodically until the bootup/setup is complete. 
> Once there is just a red light for a period, the Pi is ready.


Once your setup is configured in this manner, Pi Master should be able to ssh 
into each node via its hostname. For example, if one of our workers is 
`red001`, we may ssh to them as follows:

```
(ENV3) pi@masterpi:~ $ ssh pi@red001.local
```

If this is successful, you are ready to connect your workers to the internet.

Step 2. Configuring our Bridge

At this point, the master pi can talk to the workers via the network switch. However, these
burned Pis do not have internet access. It can be very tedious to connect each Pi individually to our WiFi. So we provide a command to "bridge" internet access between the burner Pi and the burned Pis. This program should already be installed by the cloudmesh installation script.

We can easily create our bridge as follows. 

```
(ENV3) pi@masterpi:~ $ cms bridge create --interface='wlan0'
```

This will take a moment while the dependencies are installed...

> Note the `--interface` option indicates the interface 
> used by the master pi to access the internet. 
> In this case, since we are using WiFi, it is most 
> likely `wlan0`. Other options such as `eth0` and `eth1` 
> exist for ethernet connections.

Once the installations are complete, let us restart the bridge to reflect these changes.

```
(ENV3) pi@masterpi:~ $ cms bridge restart --background
```

> Note the use of `--background` in this case is 
> recommended as the process may potentially break a 
> user's SSH pipeline (due to WiFi). If this is the case, 
> the program will continue in the background without error 
> and the user will be able to SSH shortly after.

Once the process is complete, we can use the following command to list our connected devices.

```
(ENV3) pi@masterpi:~ $ cms bridge info
bridge info

# ----------------------------------------------------------------------
#
# IP range: 10.1.1.2 - 10.1.1.122
# Manager IP: 10.1.1.1
#
# # LEASE HISTORY #
# 2021-01-21 06:04:08 dc:a6:32:e8:01:a3 10.1.1.84 red001 01:dc:a6:32:e8:01:a3
# 2021-01-21 06:04:08 dc:a6:32:e7:f0:fb 10.1.1.12 red003 01:dc:a6:32:e7:f0:fb
# 2021-01-21 06:04:08 dc:a6:32:e8:02:cd 10.1.1.22 red004 01:dc:a6:32:e8:02:cd
# 2021-01-21 06:04:08 dc:a6:32:e8:06:21 10.1.1.39 red002 01:dc:a6:32:e8:06:21
# ----------------------------------------------------------------------
```

At this point, our workers should have internet access. Let us SSH into one and ping google.com to verify.

```
(ENV3) pi@masterpi:~ $ ssh red001

pi@red001:~ $ ping google.com
PING google.com (142.250.64.238) 56(84) bytes of data.
64 bytes from mia07s57-in-f14.1e100.net (142.250.64.238): icmp_seq=1 ttl=106 time=48.2 ms
64 bytes from mia07s57-in-f14.1e100.net (142.250.64.238): icmp_seq=2 ttl=106 time=48.3 ms
64 bytes from mia07s57-in-f14.1e100.net (142.250.64.238): icmp_seq=3 ttl=106 time=47.9 ms
64 bytes from mia07s57-in-f14.1e100.net (142.250.64.238): icmp_seq=4 ttl=106 time=47.10 ms
64 bytes from mia07s57-in-f14.1e100.net (142.250.64.238): icmp_seq=5 ttl=106 time=48.5 ms
^C
--- google.com ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 9ms
rtt min/avg/max/mdev = 47.924/48.169/48.511/0.291 ms
```

Note how we are able to omit the pi user and .local extension

The cluster is now complete. For information on rebooting clusters (ie. if you shut it down for the day and wish to reboot), see [FAQ/Hints](#faqhints)

## Quickstart Guide for Mesh Networks 

This section is still under development.

In case you have a Mesh Network, the setup can typically be even more
simplifies as we can attach the unmanaged router directly to a Mesh
node via a network cable. IN that case the node is directly connected
to the internet and uses the DHCP feature from the Mesh router (see
Figure 2).

![](https://github.com/cloudmesh/cloudmesh-pi-burn/raw/main/images/network-mesh.png)

Figure 2: Networking with Mesh network

You will not need the bridge command to setup the network.

## Set up of the SSH keys and SSH tunnel

This section is still under development.

One important aspect of the cluster is to setup authentication 
with ssh, so we can easily login from the Laptop to each of the PI 
workers and the PI manager. Furthermore, we like to be able to 
login from the PI Master to each of the workers. In addition, 
we like to be able to login between the workers.

To simplify the setup of this we have developed a command 
`cms host` with the options. To generate on multiple machines 
on the network keys, to gather them and to redistribute or 
scatter them so that we can easily authenticate as discussed 
previously.

One important part of this is that the key on the Laptop must 
not be password less. This is also valid for any machine that is directly 
added to the network such as in the Mesh notwork. 

To avoid password less keys we recommend you to use `ssh-add` 
or `ssh-keychain`.

More infor and a concrete example will be documented here shortly.

The manual page for `cms host` is provided in the Manual 
Page section.

**Step 1.** On the master create ssh keys for each of the workers.

```
(ENV3) pi@masterpi:~ $ cms host key create red00[1-3]
```

**Step 2.** On the master gather all worker and master ssh keys into a file.

```
(ENV3) pi@masterpi:~ $ cms host key gather red00[1-3] keys.txt
```

**Step 3.** On the master scatter the keys to all the workers and master ~/.ssh/authorized_hosts file

```
(ENV3) pi@masterpi:~ $ cms host key scatter red00[1-3],localhost keys.txt
```

**Step 4.** Remove undeeded keys.txt file

```
(ENV3) pi@masterpi:~ $ rm keys.txt
```

**Step 5.** Verify SSH reachability from worker to master and worker to worker.

```
(ENV3) pi@masterpi:~ $ ssh red001
pi@red001:~ $ ssh masterpi  #bug if master is still named raspberrypi then the worker might resolve it as 127.0.0.1. Use raspberrypi.local instead.
(ENV3) pi@masterpi:~ $ exit
pi@red001:~ $ ssh red002
pi@red002:~ $ exit
pi@red001:~ $ exit
```

## Manual Pages

### Manual Page for the `burn` command

Note to execute the command on the commandline you have to type in
`cms burn` and not jsut `burn`.

<!--MANUAL-BURN-->
```
ModuleNotFoundError: No module named 'cloudmesh.mongo'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/home/green/Desktop/cm/cloudmesh-cmd5/cloudmesh/shell/shell.py", line 159, in <module>
    Plugin.load()
  File "/home/green/Desktop/cm/cloudmesh-cmd5/cloudmesh/shell/shell.py", line 146, in load
    commands = [pydoc.locate(x) for x in classes]
  File "/home/green/Desktop/cm/cloudmesh-cmd5/cloudmesh/shell/shell.py", line 146, in <listcomp>
    commands = [pydoc.locate(x) for x in classes]
  File "/usr/lib/python3.8/pydoc.py", line 1632, in locate
    nextmodule = safeimport('.'.join(parts[:n+1]), forceload)
  File "/usr/lib/python3.8/pydoc.py", line 362, in safeimport
    raise ErrorDuringImport(path, sys.exc_info())
pydoc.ErrorDuringImport: problem in cloudmesh.google.googlebigquery.command.googlebigquery - ModuleNotFoundError: No module named 'cloudmesh.mongo'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/usr/lib/python3.8/pydoc.py", line 347, in safeimport
    module = __import__(path)
  File "/home/green/ENV38/lib/python3.8/site-packages/cloudmesh/google/googlebigquery/command/googlebigquery.py", line 4, in <module>
    from cloudmesh.google.googlebigquery.Provider import Provider
  File "/home/green/ENV38/lib/python3.8/site-packages/cloudmesh/google/googlebigquery/Provider.py", line 5, in <module>
    from cloudmesh.mongo.DataBaseDecorator import DatabaseUpdate
ModuleNotFoundError: No module named 'cloudmesh.mongo'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/home/green/ENV38/bin/cms", line 11, in <module>
    load_entry_point('cloudmesh-cmd5', 'console_scripts', 'cms')()
  File "/home/green/ENV38/lib/python3.8/site-packages/pkg_resources/__init__.py", line 489, in load_entry_point
    return get_distribution(dist).load_entry_point(group, name)
  File "/home/green/ENV38/lib/python3.8/site-packages/pkg_resources/__init__.py", line 2852, in load_entry_point
    return ep.load()
  File "/home/green/ENV38/lib/python3.8/site-packages/pkg_resources/__init__.py", line 2443, in load
    return self.resolve()
  File "/home/green/ENV38/lib/python3.8/site-packages/pkg_resources/__init__.py", line 2449, in resolve
    module = __import__(self.module_name, fromlist=['__name__'], level=0)
  File "/home/green/Desktop/cm/cloudmesh-cmd5/cloudmesh/shell/shell.py", line 173, in <module>
    Plugin.load()
  File "/home/green/Desktop/cm/cloudmesh-cmd5/cloudmesh/shell/shell.py", line 146, in load
    commands = [pydoc.locate(x) for x in classes]
  File "/home/green/Desktop/cm/cloudmesh-cmd5/cloudmesh/shell/shell.py", line 146, in <listcomp>
    commands = [pydoc.locate(x) for x in classes]
  File "/usr/lib/python3.8/pydoc.py", line 1632, in locate
    nextmodule = safeimport('.'.join(parts[:n+1]), forceload)
  File "/usr/lib/python3.8/pydoc.py", line 362, in safeimport
    raise ErrorDuringImport(path, sys.exc_info())
pydoc.ErrorDuringImport: problem in cloudmesh.google.googlebigquery.command.googlebigquery - ModuleNotFoundError: No module named 'cloudmesh.mongo'
```
<!--MANUAL-BURN-->









### Manual Page for the `bridge` command

Note to execute the command on the commandline you have to type in
`cms bridge` and not jsut `bridge`.

<!--MANUAL-BRIDGE-->
```
ModuleNotFoundError: No module named 'cloudmesh.mongo'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/home/green/Desktop/cm/cloudmesh-cmd5/cloudmesh/shell/shell.py", line 159, in <module>
    Plugin.load()
  File "/home/green/Desktop/cm/cloudmesh-cmd5/cloudmesh/shell/shell.py", line 146, in load
    commands = [pydoc.locate(x) for x in classes]
  File "/home/green/Desktop/cm/cloudmesh-cmd5/cloudmesh/shell/shell.py", line 146, in <listcomp>
    commands = [pydoc.locate(x) for x in classes]
  File "/usr/lib/python3.8/pydoc.py", line 1632, in locate
    nextmodule = safeimport('.'.join(parts[:n+1]), forceload)
  File "/usr/lib/python3.8/pydoc.py", line 362, in safeimport
    raise ErrorDuringImport(path, sys.exc_info())
pydoc.ErrorDuringImport: problem in cloudmesh.google.googlebigquery.command.googlebigquery - ModuleNotFoundError: No module named 'cloudmesh.mongo'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/usr/lib/python3.8/pydoc.py", line 347, in safeimport
    module = __import__(path)
  File "/home/green/ENV38/lib/python3.8/site-packages/cloudmesh/google/googlebigquery/command/googlebigquery.py", line 4, in <module>
    from cloudmesh.google.googlebigquery.Provider import Provider
  File "/home/green/ENV38/lib/python3.8/site-packages/cloudmesh/google/googlebigquery/Provider.py", line 5, in <module>
    from cloudmesh.mongo.DataBaseDecorator import DatabaseUpdate
ModuleNotFoundError: No module named 'cloudmesh.mongo'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/home/green/ENV38/bin/cms", line 11, in <module>
    load_entry_point('cloudmesh-cmd5', 'console_scripts', 'cms')()
  File "/home/green/ENV38/lib/python3.8/site-packages/pkg_resources/__init__.py", line 489, in load_entry_point
    return get_distribution(dist).load_entry_point(group, name)
  File "/home/green/ENV38/lib/python3.8/site-packages/pkg_resources/__init__.py", line 2852, in load_entry_point
    return ep.load()
  File "/home/green/ENV38/lib/python3.8/site-packages/pkg_resources/__init__.py", line 2443, in load
    return self.resolve()
  File "/home/green/ENV38/lib/python3.8/site-packages/pkg_resources/__init__.py", line 2449, in resolve
    module = __import__(self.module_name, fromlist=['__name__'], level=0)
  File "/home/green/Desktop/cm/cloudmesh-cmd5/cloudmesh/shell/shell.py", line 173, in <module>
    Plugin.load()
  File "/home/green/Desktop/cm/cloudmesh-cmd5/cloudmesh/shell/shell.py", line 146, in load
    commands = [pydoc.locate(x) for x in classes]
  File "/home/green/Desktop/cm/cloudmesh-cmd5/cloudmesh/shell/shell.py", line 146, in <listcomp>
    commands = [pydoc.locate(x) for x in classes]
  File "/usr/lib/python3.8/pydoc.py", line 1632, in locate
    nextmodule = safeimport('.'.join(parts[:n+1]), forceload)
  File "/usr/lib/python3.8/pydoc.py", line 362, in safeimport
    raise ErrorDuringImport(path, sys.exc_info())
pydoc.ErrorDuringImport: problem in cloudmesh.google.googlebigquery.command.googlebigquery - ModuleNotFoundError: No module named 'cloudmesh.mongo'
```
<!--MANUAL-BRIDGE-->









### Manual Page for the `host` command

Note to execute the command on the commandline you have to type in
`cms host` and not jsut `host`.

<!--MANUAL-HOST-->
```
ModuleNotFoundError: No module named 'cloudmesh.mongo'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/home/green/Desktop/cm/cloudmesh-cmd5/cloudmesh/shell/shell.py", line 159, in <module>
    Plugin.load()
  File "/home/green/Desktop/cm/cloudmesh-cmd5/cloudmesh/shell/shell.py", line 146, in load
    commands = [pydoc.locate(x) for x in classes]
  File "/home/green/Desktop/cm/cloudmesh-cmd5/cloudmesh/shell/shell.py", line 146, in <listcomp>
    commands = [pydoc.locate(x) for x in classes]
  File "/usr/lib/python3.8/pydoc.py", line 1632, in locate
    nextmodule = safeimport('.'.join(parts[:n+1]), forceload)
  File "/usr/lib/python3.8/pydoc.py", line 362, in safeimport
    raise ErrorDuringImport(path, sys.exc_info())
pydoc.ErrorDuringImport: problem in cloudmesh.google.googlebigquery.command.googlebigquery - ModuleNotFoundError: No module named 'cloudmesh.mongo'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/usr/lib/python3.8/pydoc.py", line 347, in safeimport
    module = __import__(path)
  File "/home/green/ENV38/lib/python3.8/site-packages/cloudmesh/google/googlebigquery/command/googlebigquery.py", line 4, in <module>
    from cloudmesh.google.googlebigquery.Provider import Provider
  File "/home/green/ENV38/lib/python3.8/site-packages/cloudmesh/google/googlebigquery/Provider.py", line 5, in <module>
    from cloudmesh.mongo.DataBaseDecorator import DatabaseUpdate
ModuleNotFoundError: No module named 'cloudmesh.mongo'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/home/green/ENV38/bin/cms", line 11, in <module>
    load_entry_point('cloudmesh-cmd5', 'console_scripts', 'cms')()
  File "/home/green/ENV38/lib/python3.8/site-packages/pkg_resources/__init__.py", line 489, in load_entry_point
    return get_distribution(dist).load_entry_point(group, name)
  File "/home/green/ENV38/lib/python3.8/site-packages/pkg_resources/__init__.py", line 2852, in load_entry_point
    return ep.load()
  File "/home/green/ENV38/lib/python3.8/site-packages/pkg_resources/__init__.py", line 2443, in load
    return self.resolve()
  File "/home/green/ENV38/lib/python3.8/site-packages/pkg_resources/__init__.py", line 2449, in resolve
    module = __import__(self.module_name, fromlist=['__name__'], level=0)
  File "/home/green/Desktop/cm/cloudmesh-cmd5/cloudmesh/shell/shell.py", line 173, in <module>
    Plugin.load()
  File "/home/green/Desktop/cm/cloudmesh-cmd5/cloudmesh/shell/shell.py", line 146, in load
    commands = [pydoc.locate(x) for x in classes]
  File "/home/green/Desktop/cm/cloudmesh-cmd5/cloudmesh/shell/shell.py", line 146, in <listcomp>
    commands = [pydoc.locate(x) for x in classes]
  File "/usr/lib/python3.8/pydoc.py", line 1632, in locate
    nextmodule = safeimport('.'.join(parts[:n+1]), forceload)
  File "/usr/lib/python3.8/pydoc.py", line 362, in safeimport
    raise ErrorDuringImport(path, sys.exc_info())
pydoc.ErrorDuringImport: problem in cloudmesh.google.googlebigquery.command.googlebigquery - ModuleNotFoundError: No module named 'cloudmesh.mongo'
```
<!--MANUAL-HOST-->






## FAQ and Hints

Here, we provide some usefule FAQs and hints.

### I  used the [bridge command](#quickstart-for-restricted-wifi-access) during quickstart. How do I restart my cluster to preserve the network configuration?

> Restarting the cluster is an inevitable task. Perhaps you need to
> remove the cluster from your workspace, or you simply wish to save
> on power. This is perfectly fine. However, to preserve the network
> configuration provided by the bridge command, you should only boot
> up your workers **after** your master has finished booting. This is
> so that the `bridge` program can boot up and be operational before
> the workers attempt to establish a connection. If the workers
> establish a connection with the master before the `bridge` program
> is active, the user will have no internet access for the workers.
> In this case, you may also resolve this issue by simply rebooting
> your workers.

### Can I use the LEDs on the PI Motherboard?

> Typically this LED is used to communicate some system related
> information. However `cms pi` can controll it to switch status on
> and off. This is helpful if you like to showcase a particular state
> in the PI. Please look at the manual page. An esample is
> 
> ```bash
> $ cms pi led red off HOSTNAME
> ```
>
> that when executed on the PI (on which you also must have cms
> installed you switch the red LED off. For more options see the
> manual page


### How can I use pychar, to edit files or access files in general from my Laptop on the PI?

> This is easily possible with the help of SSHFS. To install it we
> refer you to See also: <https://github.com/libfuse/sshfs> SSHFS: add
> master to `.ssh/config` onlocal machine
>
> Let us assume you like to edit fles on a PI that you named `red`
>
> Please craete a `./.ssh/config file that containes the following:
>
> ```
>  Host red
>       HostName xxx.xxx.xxx.xxx
>       User pi
>       IdentityFile ~/.ssh/id_rsa.pub
> ```
> 
> Now let us create a directory in which we mount the remote PI directories
>
> ```
> mkdir master
> sshfs master: master -o auto_cache
> ```


