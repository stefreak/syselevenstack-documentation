# SysEleven Stack Object Storage

[TOC]

## Übersicht

SysEleven Stacks Object Service stellt einen S3 kompatiblen Object Storage dar.

Die OpenStack API kann genutzt werden um Credentials für unseren Object Storage zu erstellen. Die S3 API kann mit diversen S3 Clients und SDKs angesprochen werden.

## FAQ

### Kann ich im SysEleven Stack Object Storage (S3) verwenden?

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
