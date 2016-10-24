# SysEleven Stack Compute Service

[TOC]

## Overview

SysEleven Stacks Compute Service is built on the OpenStack Nova Projekt.

Sowohl via unserer öffentlichen OpenStack API, als auch durch das SysEleven Stack Dashboard können Compute Instanzen verwaltet werden.

## Instance types

Currently we provide you with 4 different instance types.

Name      | API Name  | Memory | VCPUs | Storage* | Network Performance
----------|-----------|--------|-------|----------|--------------------
M1 Micro  | m1.micro  | 2GB    | 1     | 50GB     | niedrig
M1 Small  | m1.small  | 8GB    | 2     | 50GB     | niedrig
M1 Medium | m1.medium | 16GB   | 4     | 50GB     | mittel
M1 Large  | m1.large  | 32GB   | 8     | 50GB     | hoch

*The storage of every instance can be extended using our Block Storage Service.

## FAQ

### Can I allocate a fixed IP to a compute instance?

Normally a fixed IP shouldn't play a big role in a cloud setup, this the infrastructure might change a lot.
If you need a fixed IP, you can assign a port from our networking service as a fixed IP to our compute instance.
For example, this is how you would use our orchestration service to fetch a fixed IP for use in your template.

``` 
  management_port:
    type: OS::Neutron::Port
    properties:
      network_id: { get_resource: management_net }
      fixed_ips:
        - ip_address: 192.168.122.100
```

### My compute instace was created, but is not accessible via SSH/HTTP etc.

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

### Kann ich eigene Images in OpenStack nutzen?
Das ist kein Problem; eigene Images können wie hier im Beispiel hochgeladen und danach genutzt werden:

```
glance  image-create --progress --is-public False --disk-format=qcow2\
 --container-format=bare --property architecture=x86_64 --name="Debian Jessie 64-bit"\
 --location http://cdimage.debian.org/cdimage/openstack/testing/debian-testing-openstack-amd64.qcow2
```
