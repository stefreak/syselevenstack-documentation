# Häufig gestellte Fragen

[TOC]

## Vorsicht beim manuellen Umhängen einer FloatingIP

Es wird ein Stack mit dem Namen Stack01 erstellt, der unter anderem eine FloatingIP Float01 benutzt. Auch diese wird im Heat-Template definiert. Nach einer Weile entsteht ein Problem auf Stack01 und es wird ein neuer (gleicher) Stack Stack02 erstellt. Einem Server von Stack02 wird über das Webinterface die FloatingIP Float01 von Stack01 zugewiesen, da ein DNS-Eintrag dafür existiert und man nicht warten kann, bis der DNS-Eintrag überall auf die neue IP zeigen würde. Das funktioniert auch ohne offensichtliche Probleme. Stack01 wird für spätere Analysen nicht gelöscht. 

Dabei gibt es aber ein Problem: Durch die Erstellung per Heat-Template ist die FloatingIP Float01 weiterhin Stack01 zugeordnet und wird zusammen mit Stack01 gelöscht, wenn dieser entfernt wird. Dies kann man nur verhindern, wenn die FloatingIP aus dem Heat-Template entfernt wird und ein "heat stack-update" auf dem Stack ausgeführt wird. 

Wenn ein Stack (z.B. Stack01) mit einem Heat-Template inklusive FloatingIP erstellt wird, muss diese aus dem Heat-Template entfernt werden und mit 

```
heat stack-update
```

aktualisiert werden. Andernfalls ist die FloatingIP weiterhin Bestandteil von Stack01. Wird die FloatingIP ohne ein stack-update - etwa manuell über das Webinterface - an einen anderen Stack gehangen (z.B. Stack02), wird sie zusammen mit Stack01 gelöscht, wenn dieser entfernt wird.  

## Ich benutze ein Ubuntu ab 15.xx und es fehlt die Default Route

Ubuntu benutzt seit 15.04 eine RFC-konforme Implementierung von DHCP. Das Software Defined Network schickt keine Default Route mit, daher muss diese explizit mit host_routes gesetzt werden. Wir arbeiten mit dem Hersteller an einer Lösung. Mit der folgenden Konfiguration kann das Problem umgangen werden. Hier ein Beispiel für das Subnet 10.0.0.0/24 mit 10.0.0.1 als Standard Gateway:

```
  subnet:
    type: OS::Neutron::Subnet
    properties:
      name: kickstart-subnet
      dns_nameservers:
        - 37.123.105.116
        - 37.123.105.117
      network_id: {get_resource: net}
      ip_version: 4
      host_routes:
        - { destination: 0.0.0.0/0, nexthop: 10.0.0.1 }
      gateway_ip: 10.0.0.1
      cidr: 10.0.0.0/24
      allocation_pools:
      - {start: 10.0.0.10, end: 10.0.0.250}
```

## Mein Stack lässt sich nicht mehr löschen! Was kann ich tun?

Ein Heat-Stack folgt beim Aufbau Abhängigkeiten, die Objekte untereinander in eine bestimmte Reihenfolge bringen. Die Dependencies werden beim Rückbau eines Stack ebenfalls berücksichtigt, so dass man ein Fehlschlagen des Löschens damit umgehen kann. Ist ein Stack ohne Angabe der Dependencies aufgebaut, schlägt das Löschen gerne mit einer Fehlermeldung wie dieser Fehl:

```
Resource DELETE failed: Conflict: resources.router_subnet_connect: Router interface for subnet eaa5a91f-3f45-43cf-8714-95118aabc64c on router 487a984c-692c-4d45-80d2-2e0ee92b505d cannot be deleted, as it is required by one or more floating IPs. 
```

In diesem Fall ist die saubere Lösung, die Abhängigkeiten von Hand zu löschen. Also erst die Floating-IP, die an dem Router hängt, dann den Router selbst und danach den ganzen Stack. Oft reicht es auch, den ``heat stack-delete <stackName>`` mehrfach aufzurufen.
Die Angabe im Heat-Template ``depends_on: <myOtherResourceID>`` vermeidet diese Probleme.


## Ist es möglich, ein Cinder Volume auf mehreren VMs zu verwenden? 
Nein, das geht aus technischen Gründen nicht. Ein Cinderdevice ist ein virtuelles Blockdevice und kann daher nicht gleichzeitig auf mehreren VMs verwendet werden.  

## Wie kann ich also shared Storage innerhalb eines Stacks bereitstellen? 

Grundsätzlich gibt es mehrere Möglichkeiten, u.a.: 

* NFS
* S3

Wir haben ein Beispiel für einen NFS-Server mit mehreren NFS-Clients, die sich automatisch mit diesem verbinden unter folgendem Link bereitgestellt: 

https://github.com/syseleven/heattemplates-examples/tree/master/sharedVolume

## Kann ich im SysEleven Stack Object Storage (S3) verwenden?

Ja. Folgende Voraussetzungen müssen dafür erfüllt sein:

* Um S3 im SysEleven Stack nutzen zu können müssen dem OpenStack Useraccount erweiterte Rechte zugewiesen werden. Dafür reicht eine E-Mail an cloudsupport@syeleven.de.

* Ein OpenStack Command Line Client in der Version >= 2.0.

* Ein S3 Client wie z.B. s3cmd

* Eine Anpassung der Shellumgebung bzw. openrc:

```
export OS_INTERFACE="public"
```

Sind diese Voraussetzungen erfüllt können wir uns die S3 Credentials generieren und anzeigen lassen:

```
openstack ec2 credentials create
openstack ec2 credentials list
```

Mit diesen Informationen können wir uns eine S3 Konfiguration mit folgendem Inhalt erstellen:

```
syseleven@kickstart:~$ cat .s3cfg
[default]
access_key = REPLACE ME
secret_key = REPLACE ME
host_base = s3.cloud.syseleven.net
host_bucket = %(bucket).s3.cloud.syseleven.net
use_https = True
check_ssl_certificate = True
check_ssl_hostname = False
```

Im nächsten Schritt erstellen wir einen S3 Bucket:

```
s3cmd mb s3://BUCKET_NAME
```

und befüllen diesen mit Inhalt:

```
s3cmd put test.jpg s3://BUCKET_NAME -P
```

Mit dem Schalter -P stellen wir die Datei der Öffentlichkeit zur Verfügung. Hierbei ist zu beachten, dass die Ausgabe von s3cmd eine falsche URL ausgibt wie z.B.: 

```
Public URL of the object is: http://BUCKET_NAME.s3.amazonaws.com/test.jpg
```

Ein Zugriff auf die S3 Datei ist unter der folgenden URL möglich: https://s3.cloud.syseleven.net/BUCKET_NAME/test.jpg

Damit sind wir in der Lage static assets über eine HTTP URL einzubinden.
