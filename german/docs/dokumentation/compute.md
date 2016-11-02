# SysEleven Stack Compute Service

[TOC]

## Übersicht

SysEleven Stacks Compute Service basiert auf dem OpenStack Nova Projekt.

Sowohl via unserer öffentlichen OpenStack API, als auch durch das SysEleven Stack Dashboard können Compute Instanzen verwaltet werden.

## Instanz-Typen

Aktuell bieten wir 4 verschiedene Instanz Typen.

Name      | API Name  | Memory | VCPUs | Storage* | Network Performance
----------|-----------|--------|-------|----------|--------------------
M1 Micro  | m1.micro  | 2GB    | 1     | 50GB     | niedrig
M1 Small  | m1.small  | 8GB    | 2     | 50GB     | niedrig
M1 Medium | m1.medium | 16GB   | 4     | 50GB     | mittel
M1 Large  | m1.large  | 32GB   | 8     | 50GB     | hoch

*Jeder Instanz kann via unserem Block Storage Service weiterer Speicher hinzugefügt werden.

## FAQ

### Kann ich einer Compute Instanz eine feste interne IP zuweisen?

Normalerweise spielen in einer Cloudumgebung feste IPs keine Rolle, da sich die Infrastruktur häufig ändert. <br>
Ist das nicht gewünscht, kann ich z.B. mit folgendem Heat-Template via unserem SysEleven Stack Orchestration Service einer Maschine eine statische IP zuweisen:

``` 
  management_port:
    type: OS::Neutron::Port
    properties:
      network_id: { get_resource: management_net }
      fixed_ips:
        - ip_address: 192.168.122.100
```

Die Konsistenz der Netzwerkarchitektur muss ich dann allerdings selbst sicherstellen.

### Meine virtuelle Maschine wurde erstellt, ist aber nicht per SSH/ HTTP etc. erreichbar.

Grundsätzlich sind alle Compute Instanzen im SysEleven Stack mit einer Default-Security-Group gesichert, die außer ICMP-Paketen keinen Traffic auf die VMs akzeptiert. Für jeden Service, der erreichbar sein soll, muss also eine Security-Group-Regel erstellt werden, die den Zugriff ermöglicht. Hier ein Beispiel wie HTTP(S)-Traffic mit einem Heat-Template unseres Orchestration Service zu ihrer Instanz erlaubt werden kann:

```
resources:
  allow_webtraffic:
    type: OS::Neutron::SecurityGroup
    properties:
      description: allow incoming webtraffic from anywhere.
      name: allow webtraffic
      rules: 
        - { direction: ingress, remote_ip_prefix: 0.0.0.0/0, port_range_min: 80, port_range_max: 80, protocol: tcp }
        - { direction: ingress, remote_ip_prefix: 0.0.0.0/0, port_range_min: 443, port_range_max: 443, protocol: tcp }
```

Diese so gebaute Security-Group muss noch an einen Port gebunden werden:

```
  example_port:
    type: OS::Neutron::Port
    properties:
      security_groups: [ get_resource: allow_webtraffic, default ]
      network_id: { get_resource: example_net}
```
Die Security-Group "default" ist in diesem Beispiel hinzugefügt, da diese Gruppe im SysEleven Stack dafür sorgt, dass Traffic, der ausgehend erlaubt ist, auch eingehend erlaubt wird.

### Kann ich eigene Images im SysEleven Stack nutzen?
Das ist kein Problem; eigene Images können hochgeladen und danach genutzt werden.
Dies geht über das [Dashboard](https://dashboard.cloud.syseleven.net/horizon/project/), [OpenStack-CLI](http://docs.openstack.org/user-guide/common/cli-install-openstack-command-line-clients.html) und als Heat-Template

#### Dashboard
Zuerst loggen wir uns im [Dashboard](https://dashboard.cloud.syseleven.net/horizon/project/) ein <br>
Nun navigieren wir zu Compute -> Images <br>
Hier angekommen klicken wir oben rechts auf "Create Image"<br>
Im sich nun öffnenden Fenster haben wir verschiedene Eingabe-Optionen: <br>

 *  Name = Der Name des Projekts
 *  Description = Beschreibung falls gewünscht
 *  Image Source = Es kann unterschieden werden zwischen "Image Location" in dem Fall kann man im nachfolgenden Feld eine externe Quelle (http,https) angeben oder "Image File" dort kann man dann im nächsten Feld, ein lokales Image nutzen
 *  Image Location/Image File = siehe c
 *  Format = Format des Images (z.B. Raw )
 *  Architecture = Architektur des Images (z.B. x86_64)
 *  Minimum Disk (optional) = Wenn angegeben, die Mindestgröße des Speicherplatzes um das Image zu booten
 *  Minimum Ram (optional) = Wenn angegeben, die Mindestgröße des vorhandenen Arbeitsspeichers 
 *  Copy Data =
 *  Public = Wenn angehakt, steht es allen Nutzern des SysEleven Stack zur Verfügung 
 *  Protected = Wenn angehakt, ist das Image nicht löschbar 

 
Sind nun alle Felder ausgefüllt, klicken wir auf "Create Image" und laden unser eigenes Image in den SysEleven Stack
 
#### OpenStack-CLI
Hierfür ist es notwendig [OpenStack-CLI](http://docs.openstack.org/user-guide/common/cli-install-openstack-command-line-clients.html) installiert zu haben.
Nun können wir mit unserer [gesourcten openrc.sh](https://doc.syselevenstack.com/tutorials/02-kickstart/#zugriff-auf-die-syseleven-stack-api) unser Image hochladen und zwar mit folgendem Befehl:
```
openstack image create Debian --disk-format qcow2 --container-format bare\  
--location http://cdimage.debian.org/cdimage/openstack/testing/debian-testing-openstack-amd64.qcow2   
```
Erklärung der einzelnen Bestandteile:

 * openstack = Der Befehl um die Openstack Tools zu nutzen
 * image = Der Befehl damit Openstack weiß, dass es um das Image geht
 * create = zum Erstellen des Images
 * Debian = Der Name des Images
 * --disk-format = Das Format des Images (ami, ari, aki, vhd, vmdk, raw, qcow2, vdi, iso)
 * --container-format = Das Format des Containers (ami, ari, aki, bare, docker, ovf)
 * --location = Der Ort des Images (Alternativ geht es auch lokal mit --file)
 
Weitere Bestandteile können mit --help angezeigt werden:
`
openstack image create --help
`
#### Heat-Template
Mit Heat kann man auch ein Image ohne weiteres hochladen. 
Ein Beispiel Template kann so aussehen:
```
heat_template_version: 2014-10-16

description: Simple template to upload an image

resources:
  glance_image:
    type: OS::Glance::Image
    properties:
      container_format: bare
      disk_format: qcow2
      name: Gentoo
      location: http://gentoo.osuosl.org/experimental/amd64/openstack/gentoo-openstack-amd64-default-20161008.qcow2
```
Eine tolle Erklärung dazu findest du hier:
https://dashboard.cloud.syseleven.net/horizon/project/stacks/resource_types/OS::Glance::Image
