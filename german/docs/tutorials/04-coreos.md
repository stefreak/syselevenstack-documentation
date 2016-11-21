# Minimales CoreOS Setup

[TOC]

## Ziel

* Wir werden eine minimale CoreOS Instanz im SysEleven Stack starten. Auf der Instanz wird ein einfacher Docker Container (nginx) laufen.
* Außerdem laden wir das aktuellste CoreOS image hoch

## Vorraussetzungen 

* Der Umgang mit einfachen Heat-Templates, wie [in den ersten Schritten](01-firststeps/) gezeigt, wird vorausgesetzt.
* Grundlagen zur Bedienung des OpenStack CLI (Umgebungsvariablen gesetzt, wie im [Kickstart-Tutorial](02-kickstart/) beschrieben.

## 1. Beispielprojekt laden

Wir arbeiten mit den Dateien im "[heattemplates-examples](https://github.com/syseleven/heattemplates-examples)" Projekt auf Github. Dieses am besten auschecken:

```
$ git clone https://github.com/syseleven/heattemplates-examples
$ cd heattemplates-examples/coreOS
```

## 2. CoreOS Image hochladen

Im "coreOS" Beispiel haben wir ein Helper-Script abgelegt um das [offizielle stabile CoreOS image](https://coreos.com/os/docs/latest/booting-on-openstack.html) in den SysEleven Stack hochzuladen.

```
$ ./upload_replacing_coreos_image.sh
```

Das Script lädt das Image herunter, löscht jegliches existierende Image mit dem Namen `private_coreos` und lädt dann das neue Image hoch.

Das Image wird im privaten Projekt-Scope abgelegt und bleibt auch dann erhalten wenn der Stack, den wir nächsten Schritt erstellen gelöscht wird.

## 3. CoreOS Instanzen starten

```
$ openstack stack create -t cluster.yaml <stack name> --parameter key_name=<ssh key name> --wait
```

key_name referenziert einen SSH-Key, der wie im [Erste-Schritte-Tutorial](01-firststeps/#ssh-key-hinterlegen) angelegt wurde.

Der Optionale Parameter `number_instances` kann angegeben werden, wenn mehr als eine Instanz hochgefahren werden soll.

## Conclusio: Was haben wir erreicht?

Jetzt sollten eine oder mehrere CoreOS Instanzen auf dem SysEleven Stack laufen. Jede ist durch eine öffentliche Floating-IP erreichbar.

`$ openstack server list` gibt eine Liste aller Instanzen sowie die jeweiligen IP-Adressen aus.

Ein Nginx sollte, in einem Docker Container laufend, auf Port 80 antworten. Zum überprüfen:
`$ curl <IP-Adresse>` (Sollte uns eine Nginx-Willkommensseite anzeigen)

Der SSH Dienst wird von der Security Group (definiert in den YAML Dateien) ebenfalls freigegeben. Daher können wir uns per SSH auf jeder CoreOS Instanz einloggen:
`$ ssh core@<IP-Adresse>`

Die Anzahl der laufenden Instanzen kann jederzeit dem Bedarf angepasst werden. Um Beispielsweise von dem Standardwert (1 Instanz) auf 5 Instanzen zu skalieren reicht dieses einfache Kommando:

```
$ openstack stack update -t cluster.yaml <stack name> --parameter key_name=<ssh key name> --number_instances=5 --wait
```

Für ein hochverfügbares Cluster fehlen noch einige Dinge, die in den weiterführenden Links erläutert werden.

## Weiterführende Links/Beispiele

* [CoreOS cluster discovery](https://coreos.com/os/docs/latest/cluster-discovery.html)
* [Running a High Availability Docker Swarm](http://tech.paulcz.net/2016/01/running-ha-docker-swarm/)
