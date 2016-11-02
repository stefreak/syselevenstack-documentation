# SysEleven Stack Block Storage

[TOC]

## Overview

SysEleven Stack Block Storage Service is built on OpenStacks Cinder project.

Via our public OpenStack API, as well as through our dashboard interface you are able to manage your Block Storage Volumes and make them available to your compute instances.
You can also create snapshots from your block storages volumes, which you might use as a boot image or as a template for the creation of other block storage volumes.

## FAQ

### Is it possible to use a block storage volume across multiple VMs simultaneously?

Unfortunately this is not possible. Think of a block storage volume like a virtual hard disk - as much as the hard disk can not be connected to multiple computers at the same time, same goes with a block storage volume and multiple compute instances.

### How can I provide a shared storage inside SysEleven Stack?

There is multiple options, for instance:

 * NFS
 * S3

We built an example template for an NFS server with multiple automatically connecting NFS clients, which can be used with our orchestration service.

[https://github.com/syseleven/heattemplates-examples/tree/master/sharedVolume](https://github.com/syseleven/heattemplates-examples/tree/master/sharedVolume)
