# SysEleven Stack Compute Service

[TOC]

## Overview

SysEleven Stacks Compute Service is built on the OpenStack Nova project.

It manages the life-cycle of compute instances in your environment. Its responsibilities include spawning, scheduling and decommissioning of virtual machines on demand.

You can manage your compute instance both via our public OpenStack API endpoints, as well as using the [Dashboard](https://dashboard.cloud.syseleven.net).
## Instance types

Currently, we provide you with four different instance types.

Name      | API Name    | Memory | VCPUs | Storage* | Network Performance
----------|-------------|--------|-------|----------|--------------------
M1 Micro  | `m1.micro`  | `2GB`  | `1`   | `50GB`   | basic
M1 Small  | m1.small  | 8GB  | 2   | 50GB   | basic
M1 Medium | `m1.medium` | `16GB` | `4`   | `50GB`   | intermediate
M1 Large  | `m1.large`  | `32GB` | `8`   | `50GB`   | high

* You can extend storage using our our Block Storage Service.

## FAQ

### Can I allocate a fixed IP to a compute instance?

Normally a fixed IP shouldn't play a big role in a cloud setup, since the infrastructure might change a lot.
If you need a fixed IP, you can assign a port from our networking service as a fixed IP to our compute instance. Here is an example which shows how to use the orchestration service to fetch a fixed IP address to use in a template:

``` 
  management_port:
    type: OS::Neutron::Port
    properties:
      network_id: { get_resource: management_net }
      fixed_ips:
        - ip_address: 192.168.122.100
```

### My compute instance was created, but is not accessible via SSH/HTTP etc.

By default all compute instances of are using the "default" security group. It's settings do not allow any other packets, except of ICMP in order to be able to ping your compute instance. Any other ports needed by a given instance need to be opened by adding a rule to the security group your instance uses (i.e., SSH or HTTPS).
Here is an example that shows how you can use a heat template to allow incoming HTTP/HTTPS traffic via your security group:

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

This security group can now be connected to a port of your network:

```
  example_port:
    type: OS::Neutron::Port
    properties:
      security_groups: [ get_resource: allow_webtraffic, default ]
      network_id: { get_resource: example_net}
```

The security group "default" is added in this example, since this group is taking care of allowing outbound traffic.

### Can I use my own images in the Compute Service?
You can easily upload your own image and use it right after, like this:

```
glance  image-create --progress --is-public False --disk-format=qcow2\
 --container-format=bare --property architecture=x86_64 --name="Debian Jessie 64-bit"\
 --location http://cdimage.debian.org/cdimage/openstack/testing/debian-testing-openstack-amd64.qcow2
```
