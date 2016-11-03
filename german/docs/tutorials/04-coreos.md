# CoreOS mit erstem Container im SysEleven Stack

## Voraussetzungen

Für dieses Tutorial wird der python-openstackclient Client benötigt.

```
pip install python-openstackclient
```
Die benötigten Umgebungsvariablen um mit der OpenStack API zu kommunizieren müssen gesetzt sein, wie in [https://doc.syselevenstack.com/tutorials/02-kickstart/#umgebungsvariablen-in-die-shell-session-einlesen](https://doc.syselevenstack.com/tutorials/02-kickstart/#umgebungsvariablen-in-die-shell-session-einlesen) beschrieben.

Ihr SSH-Key sollte im SysEleven Stack hochgeladen worden sein.

### CoreOS Im SysEleven Stack starten

Als ersten Schritt müssen wir das CoreOS Image hochladen.

```
eval $(curl https://stable.release.core-os.net/amd64-usr/current/version.txt)
wget https://stable.release.core-os.net/amd64-usr/$COREOS_VERSION/coreos_production_openstack_image.img.bz2
openstack image create --container-format bare --disk-format qcow2 --file coreos_production_openstack_image.img.bz2 "CoreOS $COREOS_VERSION"
+------------------+------------------------------------------------------+
| Field            | Value                                                |
+------------------+------------------------------------------------------+
| checksum         | 0244e1b3420b7c170e27197eed0d0025                     |
| container_format | bare                                                 |
| created_at       | 2016-11-03T16:01:11Z                                 |
| disk_format      | qcow2                                                |
| file             | /v2/images/0882e2a7-aa27-46e6-affe-8c701dc250f5/file |
| id               | 0882e2a7-aa27-46e6-affe-8c701dc250f5                 |
| min_disk         | 0                                                    |
| min_ram          | 0                                                    |
| name             | CoreOS 1185.3.0                                      |
| owner            | 2b64d96fb0434283822a1f88f00993f9                     |
| protected        | False                                                |
| schema           | /v2/schemas/image                                    |
| size             | 258548177                                            |
| status           | active                                               |
| tags             |                                                      |
| updated_at       | 2016-11-03T16:01:14Z                                 |
| virtual_size     | None                                                 |
| visibility       | private                                              |
+------------------+------------------------------------------------------+
```

CoreOS ist über User-Data konfigurierbar. Zum Beispiel um einen Nginx Docker Container nach dem Boot zu starten, folgendermaßen:

```
#cloud-config

coreos:
  units:
    - name: "docker-apache.service"
      command: "start"
      content: |
        [Unit]
        Description=Nginx container
        Author=Me
        After=docker.service

        [Service]
        Restart=always
        ExecStartPre=-/usr/bin/docker kill nginx
        ExecStartPre=-/usr/bin/docker rm nginx
        ExecStartPre=/usr/bin/docker pull nginx
        ExecStart=/usr/bin/docker run --rm --name nginx -p 80:80 nginx
        ExecStop=/usr/bin/docker stop nginx
```

Diese Datei können wir als [```user-data```](../img/user-data-coreos) vorerst abspeichern.

Mit o.g. Daten reicht ein Befehl

```
openstack server create --image c4e4ac2c-83db-4331-bf8c-3488aff057c8 --flavor m1.micro --user-data user-data --security-group default --nic 'net-id=aec02c14-5659-4eae-a267-501a28e672b4' --key-name 'mykey' coreos-nginx
```

um eine neue Compute Instanz mit CoreOS zu starten, welche den nginx Container starten wird und auf Port 80 erreichbar macht.

### Abschließender Funktionstest

```ip floating create ext-net``` erstellt uns eine Floating IP um den Container vom Internet erreichbar zu machen.

```ip floating add <IP> coreos-nginx``` verbindet genannte Floating IP mit zuvor gestarteter Compute Instanz.

```curl http://<IP>``` sollte uns nun die Willkommen-Seite des nginx anzeigen.
