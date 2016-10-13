# Resource Groups: vom Single-Server zu verteilten Setups 

[TOC]

## Infrastrukturtemplates vereinfachen mit ResourceGroups

* In diesem Tutorial wird gezeigt, wie Server in Gruppen zusammengefasst werden.
* Die Anzahl gleicher Server kann so über Parameter gesteuert werden

## Vorraussetzungen 

* Der grundlegende Umgang mit den OpenStack CLI-Tools wird als bekannt vorrausgesetzt.

## Wie es *nicht* sein sollte: redundanter Code

Ein Template, in dem mehr als ein Server gestartet wird, könnte in Auszügen so aussehen:

```
heat_template_version: 2014-10-16
.
.
.
resources:
  instance_one:
    type: OS::Nova::Server
    properties:
      image: cirros 
      flavor: m1.small
      name: server01
      user_data_format: RAW
      networks:
        - port: { get_resource: example_port1 }

  example_port1:
    type: OS::Neutron::Port
    properties:
      network_id: { get_resource: example_net}

  instance_two:
    type: OS::Nova::Server
    properties:
      image: cirros 
      flavor: m1.small
      name: server02
      user_data_format: RAW
      networks:
        - port: { get_resource: example_port2 }

  example_port2:
    type: OS::Neutron::Port
    properties:
      network_id: { get_resource: example_net}

```

Wie man sieht, werden Server und Ports jeweils zwei mal beschrieben. Das wird spätestens dann lästig, wenn Setups mehrere dutzend gleiche Server beinhalten. Bequemer ist es, wenn man die Server in Gruppen organisiert und die Beschreibung in eine eigene Datei auslagert, die dann den Server nur einmal beschreibt.

## Der elegante Weg: Gruppieren von Servern

Ein einfaches Szenario mit mehreren gleichen Servern verteilt sich damit auf zwei Dateien, einmal die *setup.yaml*, in der alles außer meinen Servern beschrieben ist:

```
heat_template_version: 2014-10-16 

#
# you can start this stack using the following command:
# 'openstack stack create -t setup.yaml <stackName>'
#

description: deploys a group of servers

resources:
  multiple_hosts:
    type: OS::Heat::ResourceGroup
    depends_on: example_subnet
    properties:
      count: 2 
      resource_def: 
        type: server.yaml
        properties:
          network_id: { get_resource: example_net}
          server_name: server_%index%

  example_net:
    type: OS::Neutron::Net
    properties:
      name: example-net

  example_subnet:
    type: OS::Neutron::Subnet
    properties:
      name: example_subnet
      dns_nameservers:
        - 37.123.105.116
        - 37.123.105.117
      network_id: {get_resource: example_net}
      ip_version: 4
      cidr: 10.0.0.0/24
      allocation_pools:
      - {start: 10.0.0.10, end: 10.0.0.250}
```

Die dazugehörige *server.yaml* enthält die eigentliche Definition unserer VM:

```
heat_template_version: 2014-10-16

description: single server resource used by resource groups.

parameters:
  network_id:
    type: string
  server_name:
    type: string

resources:
  my_instance:
    type: OS::Nova::Server
    properties:
      user_data_format: RAW
      image: Ubuntu 16.04 sys11-cloudimg amd64 
      flavor: m1.small
      name: { get_param: server_name }
      networks:
        - port: { get_resource: example_port }

  example_port:
    type: OS::Neutron::Port
    properties:
      network_id: { get_param: network_id }
```

## Fazit: Modularer Code

Wir erreichen so mehrere Vorteile:

* Unser Code ist modularisiert.
* Redundanz im Code wird vermieden.
* Wir haben einen automatisch fortlaufenden Index, den wir hier am Beispiel der Servernamen nutzen.
* Die Anzahl der Ressourcen in einer Gruppe lässt sich einfach in Parameter auslagern, so dass verschieden skalierte Setups aus der selben Code-Basis erstellt werden können.

 


