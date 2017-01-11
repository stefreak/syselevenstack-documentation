# Openstack CLI

[TOC]

## Übersicht

Dieses Tutorial dient als Anleitung für die Installation des Openstack CLI (Command Line Interfacs).<br>
Mit dem Openstack CLI kannst du deine Stacks administrieren und überwachen.

### Voraussetzungen

* PC/MAC
* Adminrechte
* Grundlegende PC-Kentnisse

In dieser Anleitung gehen wir davon aus, dass bisher nichts von den benötigten Programmen installiert wurde.
Für den Fall das bereits Bestandteile installiert sind, überspringe den jeweiligen Punkt.

### Installation unter MAC

Optional kann auch ein alternatives Terminal wie [iTerm2](https://www.iterm2.com/) installiert werden.

Als erstes öffnen wir das Terminal oder falls installiert iTerm2.
Damit wir die benötigten Pakete installieren können, nutzen wir das Paketverwaltungsprogramm [PIP](https://de.wikipedia.org/wiki/Pip_(Python))

Diese installieren wir mit folgendem Befehl:
```
easy_install pip
```

Nachdem wir nun PIP installiert haben können wir ganz einfach den Openstack Client installieren.

Dafür nutzen wir folgenden Befehl:
```
pip install python-openstackclient
```

### Installation unter Windows

Damit das OpenStack CLI unter Windows genutzt werden kann, benötigen wir zunächst [Python 2.7](https://www.python.org/downloads/release/python-2712/)
Nachdem die Installation abgeschlossen ist, wechseln wir in die Eingabeaufforderung und stellen sicher das wir im folgenden Pfad sind C:\Python27\Scripts

Nun nutzen wir das easy_install Kommando um [PIP](https://de.wikipedia.org/wiki/Pip_(Python)) zu installieren
```
C:\Python27\Scripts>easy_install pip
```

Nachdem nun PIP installiert ist, brauchen wir nun als letzten Schritt nur noch das OpenStack CLI installieren:
```
C:\Python27\Scripts>pip install python-openstackclient
```

### Installation unter Red Hat Enterprise Linux, CentOS oder Fedora

Um das OpenStack CLI zu installieren benötigen wir zunächst [PIP](https://de.wikipedia.org/wiki/Pip_(Python))
```
yum install python-devel python-pip
```

Nachdem wir PIP installiert haben, brauchen wir nun nur noch den OpenStack Clienten zu installieren:
```
pip install python-openstackclient
```

#### Weitere mögliche optionale Plugins

Es bestehe dir Möglichkeit weitere Plugins zu installieren, z.B. Heat.
Dafür fügen wir in den nachfolgenden Befehl einfach das entsprechende Plugin ein
```
pip install python-PLUGINclient
```

Hier einige möglichen Beispiel Plugins:

* barbican - Key Manager Service API
* ceilometer - Telemetry API
* cinder - Block Storage API and extensions
* cloudkitty - Rating service API
* designate - DNS service API
* fuel - Deployment service API
* glance - Image service API
* gnocchi - Telemetry API v3
* heat - Orchestration API
* magnum - Container Infrastructure Management service API
* manila - Shared file systems API
* mistral - Workflow service API
* monasca - Monitoring API
* murano - Application catalog API
* neutron - Networking API
* nova - Compute API and extensions
* sahara - Data Processing API
* senlin - Clustering service API
* swift - Object Storage API
* trove - Database service API

### Fazit
Nun ist der Openstack Client installiert und wir können diesen nutzen.
Dieser wird zum Beispiel im [SysEleven Stack API - Beginner Tutorial](https://doc.syselevenstack.com/de/tutorials/02-kickstart) benötigt.

Eine Übersicht aller Befehle kannst du dir mit folgendem Befehl anzeigen lassen:
```
openstack --help
```