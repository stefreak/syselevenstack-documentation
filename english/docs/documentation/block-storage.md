# SysEleven Stack Block Storage

[TOC]

## Overview

SysEleven Stack Block Storage Service is built on the OpenStack Cinder project.

The Block Storage Service adds persistent block storage to your compute instances.

You can manage your block storage volumes and make them available to your compute instances both via our public OpenStack API endpoints, as well as using the [Dashboard](https://dashboard.cloud.syseleven.net).

You can also create snapshots from your block storages volumes, which you can use as a boot image or as a template for the creation of other block storage volumes.

## FAQ

### Can I use a block storage volume across multiple VMs simultaneously?

Unfortunately this is not possible. Think of a block storage volume like a virtual hard disk - just as you cannot connect a hard disk to multiple computers at the same time, you cannnot connect a block storage volume to multiple compute instances at the same time.

### How can I get shared storage inside SysEleven Stack?

There are multiple options, for instance:

 * S3 (provided as part of the SysEleven Stack)
 * NFS (interim solution for legacy applications)

For the NFS case, we provide a [shared volume](https://github.com/syseleven/heattemplates-examples/tree/master/sharedVolume) template with multiple automatically connecting NFS clients, which you can use with our orchestration service.
