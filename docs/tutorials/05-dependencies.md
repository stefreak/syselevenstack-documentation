# Defining dependiencies between openstack resources 

[TOC]

## Ziel

* In diesem Tutorial wird gezeigt, wie einzelne Resourcen in einem Stack aufeinander aufbauend organisiert werden können.
* Damit ist es möglich, beispielsweise erst die Netzwerk-Infrastruktur aufzubauen, bevor dort Server gestartet werden.

## Vorraussetzungen 

* Der Umgang mit einfachen Heat-Templates, wie in den [ersten Schritten](https://doc.syselevenstack.com/tutorials/01-firststeps/) gezeigt, wird vorausgesetzt.
* Dieses Tutorial startet einen Heat-Stack über die CLI-Tools, Vorwissen über die Bedienung des openstack CLI Client ist daher sinnvoll

## Das Problem: Der Aufbau von Ressourcen scheitert

Wenn die Reihenfolge bestimmter Ressourcen nicht vorgegeben wird, werden die Ressourcen willkürlich angelegt. Der Aufbau eines Stack schlägt dann nach folgendem Muster fehl:

```
syselevenstack@kickstart:~/failingDemo$ openstack stack create -t clustersetup.yaml -e clustersetup-env.yaml noDependencies --wait
2016-10-18 13:50:48 [noDependencies]: CREATE_IN_PROGRESS  Stack CREATE started
2016-10-18 13:50:48 [syseleven_net]: CREATE_IN_PROGRESS  state changed
2016-10-18 13:50:48 [syseleven_router]: CREATE_IN_PROGRESS  state changed
2016-10-18 13:50:50 [syseleven_net]: CREATE_COMPLETE  state changed
2016-10-18 13:50:50 [syseleven_router]: CREATE_COMPLETE  state changed
2016-10-18 13:50:50 [dbserver_group]: CREATE_IN_PROGRESS  state changed
2016-10-18 13:50:51 [servicehost_group]: CREATE_IN_PROGRESS  state changed
2016-10-18 13:50:53 [lb_group]: CREATE_IN_PROGRESS  state changed
2016-10-18 13:50:55 [appserver_group]: CREATE_IN_PROGRESS  state changed
2016-10-18 13:50:58 [syseleven_subnet]: CREATE_IN_PROGRESS  state changed
2016-10-18 13:51:00 [syseleven_subnet]: CREATE_COMPLETE  state changed
2016-10-18 13:51:00 [router_subnet_connect]: CREATE_IN_PROGRESS  state changed
2016-10-18 13:51:02 [router_subnet_connect]: CREATE_COMPLETE  state changed
2016-10-18 13:51:02 [dbserver_group]: CREATE_FAILED  BadRequest: resources.dbserver_group.resources[0].resources.dbserver: Port de3404ba-3b43-4123-bf1d-1a1809ea434a requires a FixedIP in order to be used. (HTTP 400) (Request-ID: req-bcd57a8b-7290-4311-8800-46fc78fb0f8d)
2016-10-18 13:51:10 [lb_group]: CREATE_COMPLETE  state changed
2016-10-18 13:51:11 [servicehost_group]: CREATE_COMPLETE  state changed
2016-10-18 13:51:17 [appserver_group]: CREATE_COMPLETE  state changed
2016-10-18 13:51:17 [noDependencies]: CREATE_FAILED  Resource CREATE failed: BadRequest: resources.dbserver_group.resources[0].resources.dbserver: Port de3404ba-3b43-4123-bf1d-1a1809ea434a requires a FixedIP in order to be used. (HTTP 400) (Request-ID: req-bcd57a8b-7290-4311-8800-46fc78fb0f8d)

 Stack noDependencies CREATE_FAILED 

syselevenstack@kickstart:~/failingDemo$ 
```

## Die Lösung: saubere Definition der Reihenfolge

In diesem Code-Beispiel geben wir eine saubere Kette an Abhängigkeiten an, so dass das Setup sauber hoch fährt und sich auch wieder problemlos löschen lässt. Hier der Auszug aus dem verteilten Beispiel-Setup(link required):

```
resources:
  syseleven_net:
    type: OS::Neutron::Net
    properties: 
      name: syseleven-net

  syseleven_subnet:
    type: OS::Neutron::Subnet
    depends_on: [ syseleven_net ]
    properties:
      name: syseleven_subnet
      dns_nameservers:
        - 37.123.105.116
        - 37.123.105.117
      network_id: {get_resource: syseleven_net}
      ip_version: 4
      cidr: 192.168.2.0/24
      allocation_pools:
      - {start: 192.168.2.10, end: 192.168.2.250}

  syseleven_router:
    type: OS::Neutron::Router
    depends_on: [ syseleven_subnet ]
    properties:
      external_gateway_info: {"network": { get_param: public_network_id }}

  router_subnet_connect:
    type: OS::Neutron::RouterInterface
    depends_on: [ syseleven_router ]
    properties:
      router_id: { get_resource: syseleven_router }
      subnet: { get_resource: syseleven_subnet }

  ### Nodes as resource group ###
  #######################
  node_group:
    type: OS::Heat::ResourceGroup
    depends_on: [ router_subnet_connect ]
    properties:
      count: 1 
      resource_def: 
        type: server.yaml
        properties:
          name: server%index%
          image: { get_param: image }
          flavor: { get_param: flavor_server }
          syseleven_net: { get_resource: syseleven_net }
          public_network_id: { get_param: public_network_id }

```



## Im Ergebnis sehen wir, dass der Stack nach genua unseren Vorgaben zusammengesetzt und gestartet wird:

D.h. wir haben folgende Reihenfolge festgelegt:

```
   -----------------
1.  syseleven_net
   -----------------
      |
     -----------------
2.    syseleven_subnet
     -----------------
        |
       -----------------
3.      syseleven_router
       -----------------
          |
         -----------------
4.        router_subnet_connect
         -----------------
            |
           -----------------
5.          node_group 
           -----------------
```

Über den openstack CLI Client gestartet lässt sich das mit der Option --wait gut beobachten:

```
syselevenstack@kickstart:~/heattemplates-examples/example-setup$ openstack stack create -t clustersetup.yaml -e clustersetup-env.yaml depTest --wait
2016-10-19 11:04:54 [depTest]: CREATE_IN_PROGRESS  Stack CREATE started
2016-10-19 11:04:54 [syseleven_net]: CREATE_IN_PROGRESS  state changed
2016-10-19 11:04:55 [syseleven_net]: CREATE_COMPLETE  state changed
2016-10-19 11:04:55 [syseleven_subnet]: CREATE_IN_PROGRESS  state changed
2016-10-19 11:04:57 [syseleven_subnet]: CREATE_COMPLETE  state changed
2016-10-19 11:04:57 [syseleven_router]: CREATE_IN_PROGRESS  state changed
2016-10-19 11:04:59 [syseleven_router]: CREATE_COMPLETE  state changed
2016-10-19 11:04:59 [router_subnet_connect]: CREATE_IN_PROGRESS  state changed
2016-10-19 11:05:01 [router_subnet_connect]: CREATE_COMPLETE  state changed
2016-10-19 11:05:06 [node_group]: CREATE_IN_PROGRESS  state changed
```

## Beispiel-Ressourcen:

Das auf Github veröffentlichte Beispiel [examplesetup](https://github.com/syseleven/heattemplates-examples/tree/master/example-setup) setzt Dependencies ein.


