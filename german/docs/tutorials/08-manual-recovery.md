# Manuelle Datenwiederherstellung

[TOC]

## Ziel

* Herunterladen eines Instanz-Snapshots
* Dateisystem im Snapshot reparieren und mounten

## Vorraussetzungen 

* Der Umgang mit einfachen Heat-Templates, wie [in den ersten Schritten](01-firststeps/) gezeigt, wird vorausgesetzt.
* Grundlagen zur Bedienung des OpenStack CLI (Umgebungsvariablen gesetzt, wie im [Kickstart-Tutorial](02-kickstart/) beschrieben.

## Optional: Temporäre Arbeitsumgebung

Für dieses Tutorial benötigen wir eine Linux-Umgebung mit OpenStack Client. Sollte diese noch nicht vorhanden sein, kann sie mit folgenden Kommandos erstellt werden:

```
wget https://raw.githubusercontent.com/syseleven/heattemplates-examples/master/gettingStarted/sysElevenStackKickstart.yaml
...
$ openstack stack create -t sysElevenStackKickstart.yaml --parameter key_name=<ssh key name> <stack name> --wait
...
$ ssh syseleven@<server-ip>

# Zugangsdaten eintragen
$ vi openrc
```

## Erstellen eines Snapshots der defekten Instanz

Um die Daten herunterladen zu können muss ein Snapshot erstellt werden.

Achtung: Der betroffene Server wird für wenige Minuten angehalten.

```
$ openstack server image create <server uuid> --name <snapshot name> --wait
```

Nun kann der Snapshot heruntergeladen werden. Dies kann eine Weile dauern.

```
$ openstack image save --file snapshot.qcow2 <snapshot name>
```

Auf das Dateisystem kann nun mit nbd:

```
$ sudo apt-get install -y qemu-utils
$ sudo modprobe nbd
$ sudo qemu-nbd --connect /dev/nbd0 snapshot.qcow2
```

Nun können die verfügbaren Partitionen angezeigt werden.

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

Mit fsck können etwaige Fehler erkannt und repariert werden.

```
# Mit der Option -y wird ohne Rückfrage repariert.
$ sudo fsck -f /dev/nbd0p1
[...]
```

Jetzt kann das Dateisystem eingehängt werden.

```
$ sudo mount /dev/nbd0p1 /mnt/
```

Die Daten sind nun zugreifbar unter `/mnt/`. Bei ext-Dateisystemen sollte ein Blick in `/mnt/lost+found` geworfen werden.