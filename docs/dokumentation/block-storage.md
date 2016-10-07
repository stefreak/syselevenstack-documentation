# SysEleven Stack Block Storage

[TOC]

## Übersicht

SysEleven Stacks Block Storage Service basiert auf dem OpenStack Cinder Projekt.

Sowohl via unserer öffentlichen OpenStack API, als auch durch das SysEleven Stack Dashboard können Block Storage Volumes verwaltet und in Compute Instanzen verfügbar gemacht werden.
Es können Snapshots von Block Storage Volumes erstellt werden, welche als Grundlage zur Erstellung neuer Volumes oder Images genutzt werden können.

## FAQ

### Ist es möglich, ein Cinder Volume auf mehreren VMs zu verwenden? 

Nein, das geht aus technischen Gründen nicht. Ein Cinderdevice ist ein virtuelles Blockdevice und kann daher nicht gleichzeitig auf mehreren VMs verwendet werden.  

### Wie kann ich also shared Storage innerhalb eines Stacks bereitstellen? 

Grundsätzlich gibt es mehrere Möglichkeiten, u.a.: 

* NFS
* S3

Wir haben ein Beispiel für einen NFS-Server mit mehreren NFS-Clients, die sich automatisch mit diesem verbinden unter folgendem Link bereitgestellt, welcher mit unserem Orchestration Service verwendet werden kann: 

https://github.com/syseleven/heattemplates-examples/tree/master/sharedVolume
