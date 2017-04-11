# Manual data recovery

[TOC]

## Goal

* Downloading a snapshot of an instance
* Repairing the filesystem and mount it to extract data

## Prerequisites

* You should be able to use simple heat templates, like shown in the [first steps tutorial](01-firststeps/).
* You know the basics of using the OpenStack CLI (Environment variables are set, like shown in the [kick-start tutorial](02-kickstart/).

## Optional: temporary work environment

For this tutorial, we need a *Linux* environment and the OpenStack client.

If you do not have that yet, you can create it with the following commands:

```
wget https://raw.githubusercontent.com/syseleven/heattemplates-examples/master/gettingStarted/sysElevenStackKickstart.yaml
...
$ openstack stack create -t sysElevenStackKickstart.yaml --parameter key_name=<ssh key name> <stack name> --wait
...
```

Now we need to connect to the created instance.

```
$ ssh syseleven@<server-ip>
```

The following commands need to be executed in the ssh session.

We also need the OpenStack credentials (openrc-file).
You can download the file here: https://dashboard.cloud.syseleven.net/horizon/project/access_and_security/api_access/openrc/

```
$ source openrc
```

## Create a snapshot

To recover data from an existing instance, we have to create a snapshot first.

Attention: The server will be unavailable for a few minutes.

```
$ openstack server image create <server uuid> --name <snapshot name> --wait
```

We can download the snapshot now. This can take a while.

```
$ openstack image save --file snapshot.qcow2 <snapshot name>
```

We can access the snapshot's contents via nbd.

```
$ sudo apt-get install -y qemu-utils
$ sudo modprobe nbd
$ sudo qemu-nbd --connect /dev/nbd0 snapshot.qcow2
```

Let's list the partitions.

```
$ sudo fdisk -l /dev/nbd0
Disk /dev/nbd0: 50 GiB, 53687091200 bytes, 104857600 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x974bb19a

Device      Boot Start       End   Sectors Size Id Type
/dev/nbd0p1 *     2048 104857566 104855519  50G 83 Linux
```

To repair the filesystem, use fsck.

```
# Using the option -y, fsck will repair without asking.
$ sudo fsck -f /dev/nbd0p1
[...]
```

Now we can mount the filesystem.

```
$ sudo mount /dev/nbd0p1 /mnt/
```

# Conclusion

Now the data is accessible on `/mnt/`. If it's an ext filesystem you should have a look in `/mnt/lost+found`.