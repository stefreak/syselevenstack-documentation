# SysEleven Stack Orchestration Service

[TOC]

The SysElevenStack orchestration service is built on the OpenStack Heat project.

It orchestrates multiple composite cloud applications by using the OpenStack native HOT template format. It can be seen as an overlay template specification, which through a native REST API can configure the underlying SysEleven Stack services.

You can use the orchestration service both via our public OpenStack API endpoints, as well as using the [Dashboard](https://dashboard.cloud.syseleven.net)

With the orchestration service you can do more than just start and stop virtual machines: You have control over your stack's network, storage, security groups, as well as your virtual machines. To run web services successcully on the SysEleven Stack, these components need to be known and orchestrated.

## Network

It is technically possible to run virtual machines without a network, but the majority of applications need connectivity. In OpenStack there are five kinds of objects that are necessary to define a network: *Networks*, *Subnets*, *Ports*, *Routers*, as well as *Floating IPs*.

*Networks* are a kind of container for one or more *Subnets* A *Subnet* is the network actually used by a stack to route traffic from and to the outside world. A virtual machine without a *Subnet* will not be able to talk to the outside world.

Please keep in mind that in OpenStack everything is an *Object* or *Resource* which you can manage via an API. This means you can define a network as part of a Heat stack. As an example, you could create a file named `net.yaml` with the following content:

```
heat_template_version: 2014-10-16
resources:
  net:
    type: OS::Neutron::Net
    properties:
      name: example-net
  subnet:
    type: OS::Neutron::Subnet
    properties:
      name: example-subnet
      dns_nameservers:
        - 37.123.105.116
        - 37.123.105.117
      network_id: {get_resource: net}
      ip_version: 4
      cidr: 10.0.0.0/24
      allocation_pools:
      - {start: 10.0.0.10, end: 10.0.0.250}
```

Now, create this network by running

```
heat stack-create -f net.yaml netexample1
```

If you now open the [Dashboard](https://dashboard.cloud.syseleven.net), you will see that you just made a piece of infrastructure: In the *Network* tab you see the network and subnet as defined in `net.yaml`. We do not need this network, so we clean it up with the following command:

```
heat stack-delete netexample1
```

Aside from *Networks* and *Subnets*, *Routers* are basic building blocks of an infrastructure stack. You need routers to connect your subnets to the Internet. This way you enable your virtual machines to pull updates from an upstream source, for example. You also need *Routers* so clients can reach your virtual machines from the internet. For traffic to the Internet, we use Source Network Address Translation to assign IPv4 based network traffic to the correct virtual machine.

You can now expand the previous example and add a *Router*:

```
heat_template_version: 2014-10-16
parameters:
  public_network_id:
    type: string
resources:
  net:
    type: OS::Neutron::Net
    properties:
      name: example-net
  subnet:
    type: OS::Neutron::Subnet
    properties:
      name: example-subnet
      dns_nameservers:
        - 37.123.105.116
        - 37.123.105.117
      network_id: {get_resource: net}
      ip_version: 4
      cidr: 10.0.0.0/24
      allocation_pools:
      - {start: 10.0.0.10, end: 10.0.0.250}
  router:
    type: OS::Neutron::Router
    properties:
      external_gateway_info: {"network": { get_param: public_network_id }}
      name: example-router
```

This example is the first where you need a parameter: The `public_network_id`. You can get a list of available networks with public IP addresses like this:

```
syselevenstack@kickstart:~$ neutron net-list
+--------------------------------------+---------------+---------------------------------------------------+
| id                                   | name          | subnets                                           |
+--------------------------------------+---------------+---------------------------------------------------+
| 02fc43b8-6de5-4e26-8bc7-7e70f0f3ca1a | float2        | 6c9e0e07-f7ac-40e3-b208-febd9d8cd0b8              |
| 4f996f76-e943-4e91-bfe2-d01b00283d86 | kickstart-net | d134c951-aaa2-4c9b-9cac-ae51b96f5533 10.0.0.0/24  |
| 80ca1837-a461-4621-b58d-79507aa8b044 | float1        | d79b58c4-23f3-476b-82f2-e00e348d25d4              |
+--------------------------------------+---------------+---------------------------------------------------+
```

Choose one of the public networks, for this example **float1** with the ID `80ca1837-a461-4621-b58d-79507aa8b044`. Again, create the network, just with a parameter:
```
heat stack-create -f net2.yaml \
                  -P public_network_id=80ca1837-a461-4621-b58d-79507aa8b044 \
                  netexample2
```

You can check in the [Dashboard](https://dashboard.cloud.syseleven.net) under "Network Topology" to see that the object was created correctly. You can also see that *Network* and *Router* are independent objects. To connect both objects you need an additional object that does just that: A *Router-Subnet-Connect*. Here is the code to add this piece of infrastructure:

```
parameters:
  public_network_id:
    type: string
resources:
  net:
    type: OS::Neutron::Net
    properties:
      name: example-net
  subnet:
    type: OS::Neutron::Subnet
    properties:
      name: example-subnet
      dns_nameservers:
        - 37.123.105.116
        - 37.123.105.117
      network_id: {get_resource: net}
      ip_version: 4
      cidr: 10.0.0.0/24
      allocation_pools:
      - {start: 10.0.0.10, end: 10.0.0.250}
  router:
    type: OS::Neutron::Router
    properties:
      external_gateway_info: {"network": { get_param: public_network_id }}
      name: example-router
  router_subnet_bridge:
    type: OS::Neutron::RouterInterface
    depends_on: subnet
    properties:
      router_id: { get_resource: router }
      subnet: { get_resource: subnet }
```

If you start this template with the following command:

```
heat stack-create -f net3.yaml \
                  -P public_network_id=80ca1837-a461-4621-b58d-79507aa8b044 \
                  netexample3
```

You can see in the [Dashboard](https://dashboard.cloud.syseleven.net) that you created a private network `example-net`, which is connected to the public network `float1` through a Router. 

Using this infrastructure we can now start a virtual machine which has an outside network connections. All that is missing is a way to assign a virtual machine to a given subnet. This is done using *Ports*. *Ports* are the network interfaces of a virtual machine: A *Port* needs to be connected to a *Subnet* for the virtual machine to be able to use it. Here is the code to connect a *Port* to a *Subnet*:

```
  port:
    type: OS::Neutron::Port
    properties:
      network_id: { get_resource: net}
```

Now you have defined and created the major parts of an orchestrated setup. Next, you start a virtual machine that uses the infrastructure created so far::

```
heat_template_version: 2014-10-16

parameters:
 key_name:
  type: string
 public_network_id:
  type: string

resources:

  host:
    type: OS::Nova::Server
    properties:
      name: example host
      image: Ubuntu-14.04-LTS from cloud-images.ubuntu.com
      key_name: { get_param: key_name }
      flavor: m1.micro
      networks:
        - port: { get_resource: port }

  port:
    type: OS::Neutron::Port
    properties:
      network_id: { get_resource: net}

  net:
    type: OS::Neutron::Net
    properties:
      name: example-net

  subnet:
    type: OS::Neutron::Subnet
    properties:
      name: example-subnet
      dns_nameservers:
        - 37.123.105.116
        - 37.123.105.117
      network_id: {get_resource: net}
      ip_version: 4
      cidr: 10.0.0.0/24
      allocation_pools:
      - {start: 10.0.0.10, end: 10.0.0.250}

  router:
    type: OS::Neutron::Router
    properties:
      external_gateway_info: {"network": { get_param: public_network_id }}
      name: example-router

  router_subnet_bridge:
    type: OS::Neutron::RouterInterface
    depends_on: subnet
    properties:
      router_id: { get_resource: router }
      subnet: { get_resource: subnet }
```

You can use this template as usual, only that you reference the public SSH Key you stored in the Dashboard using the command line switch `-Pkey_name=<PubKeyName>` This ensures that you can log in to the default account on your virtual machine using SSH.

In the [Dashboard](https://dashboard.cloud.syseleven.net) you can see the network being built. You also see the subnet and router are created and all objects will be connected. We cannot connect to our virtual machine though: The setup is missing a publically accessible IP address. The missing object is a *Floating IP*, another object we need to connect with our *Port*. When that's done, we have a virtual machine that is reachable from the Internet. Here is the necessary orchestration code:

```
heat_template_version: 2014-10-16

parameters:
 key_name:
  type: string
 public_network_id:
  type: string

resources:

  host:
    type: OS::Nova::Server
    properties:
      name: example host
      image: Ubuntu-14.04-LTS from cloud-images.ubuntu.com
      key_name: { get_param: key_name }
      flavor: m1.micro
      networks:
        - port: { get_resource: port }

  port:
    type: OS::Neutron::Port
    properties:
      network_id: { get_resource: net}

  net:
    type: OS::Neutron::Net
    properties:
      name: example-net

  subnet:
    type: OS::Neutron::Subnet
    properties:
      name: example-subnet
      dns_nameservers:
        - 37.123.105.116
        - 37.123.105.117
      network_id: {get_resource: net}
      ip_version: 4
      cidr: 10.0.0.0/24
      allocation_pools:
      - {start: 10.0.0.10, end: 10.0.0.250}

  router:
    type: OS::Neutron::Router
    properties:
      external_gateway_info: {"network": { get_param: public_network_id }}
      name: example-router

  router_subnet_bridge:
    type: OS::Neutron::RouterInterface
    depends_on: subnet
    properties:
      router_id: { get_resource: router }
      subnet: { get_resource: subnet }

  floating_ip:
    type: OS::Neutron::FloatingIP
    properties:
      floating_network: { get_param: public_network_id }
      port_id: { get_resource: port }
```

## Security Groups

But even then, attempts to connect to the machine will fail both in the browser or via SSH. What is missing? You need to decide how your virtual machine should be used and allow network traffic to flow accordingly. The default security policy in the SysEleven Stack forbids all traffic coming from the Internet. This is a good practice to ensure stacks are secure by default and you do not expose internal systems, i.e. a database server, to the Internet accidentally.

You can change that policy easily by adding another object: A *Security Group*. Security groups are similar to simple firewalls: You need to define the protocol, maybe a port or port range, and the source and target IP addresses or address ranges, as well as the direction of traffic. Incoming traffic is called *ingress*, outgoing traffic *egress*. 

If you now start your stack, all objects are successfully combined and your first virtual machine is live! Do not worry, future machines will be less complicated to bring up, since you will build a collection of templates that cover your use cases. Here is the full orchestration template that allows you to start the minimal example we built so far:

```
heat_template_version: 2014-10-16

parameters:
 key_name:
  type: string
 public_network_id:
  type: string

resources:

  host:
    type: OS::Nova::Server
    properties:
      name: example host
      image: Ubuntu-14.04-LTS from cloud-images.ubuntu.com
      key_name: { get_param: key_name }
      flavor: m1.micro
      networks:
        - port: { get_resource: port }

  port:
    type: OS::Neutron::Port
    properties:
      network_id: { get_resource: net}

  net:
    type: OS::Neutron::Net
    properties:
      name: example-net

  subnet:
    type: OS::Neutron::Subnet
    properties:
      name: example-subnet
      dns_nameservers:
        - 37.123.105.116
        - 37.123.105.117
      network_id: {get_resource: net}
      ip_version: 4
      cidr: 10.0.0.0/24
      allocation_pools:
      - {start: 10.0.0.10, end: 10.0.0.250}

  router:
    type: OS::Neutron::Router
    properties:
      external_gateway_info: {"network": { get_param: public_network_id }}
      name: example-router

  router_subnet_bridge:
    type: OS::Neutron::RouterInterface
    depends_on: subnet
    properties:
      router_id: { get_resource: router }
      subnet: { get_resource: subnet }

  floating_ip:
    type: OS::Neutron::FloatingIP
    properties:
      floating_network: { get_param: public_network_id }
      port_id: { get_resource: port }

  allow_ssh:
    type: OS::Neutron::SecurityGroup
    properties:
      description: allow incoming SSH and ICMP traffic from anywhere.
      name: allow incoming traffic, tcp port 22 and icmp
      rules:
        - { direction: ingress, remote_ip_prefix: 0.0.0.0/0, port_range_min: 22, port_range_max: 22, protocol: tcp }
        - { direction: ingress, remote_ip_prefix: 0.0.0.0/0, protocol: icmp }
```

After creation has finished, you can log in to the virtual machine. Find it's IP address with the following command:

```
nova list
```

Copy the IP address and log into the virtual machine:
```
ssh ec2-user@<IP-Adresse>
```

In Ubuntu cloud images, `ec2-user` is the default name of the default user account.

You got to know the network and virtual machine parts of orchestration. You do not need anything else to run a simple stack. But many web applications have operational constraints we did not cover yet: What happens if you need to change the size or number of our virtual machines? How do you preserve and find my data if I delete my stack as shown above? You can find answers to these questions in the [Block Storage documentation](../block-storage). Every virtual machine currently comes with 50 Gigabytes of storage. If you need additional storage, you need to create and use volumes. Volumes are also interesting from another point of view: If you want to preserve data beyond the life time of a virtual machine (for example a database for a web application), you need to use volumes. The storage that comes with a virtual machine is *ephemeral*: it is lost when the virtual machine is deleted. To provide long lasting storage, create a stack to create and provide a volume of the required storage size.

<!--- TODO: Code fehlt. -->
```
```

Using a storage template, you get a volume with a UID. You can pass this UID to a virtual machine using a parameter, where the volume will show up as an additional block device, just like an additional hard disk. Using that block device, data can be stored persistently.

You can also build a setup where a virtual machine has more than 50Gigabytes of storage. In that case you do not need to create the volume in a separate stack, you can just expand the orchestration template for your virtual machine. A complete setup would look like this:

<!--- TODO: Code fehlt. -->
```
```

## Bestandteile eines Heat-Templates.

Wie ist ein Heat-Template, mit dem ich jeden Aspekt meiner Infrastruktur
beschreiben kann, aufgebaut? Die Dateien, die die virtuelle Infrastruktur
beschreiben, sind in YAML geschrieben. Sie sind unterteilt in fünf Sektionen:
`heat_template_version`, `description`, `parameters`, `resources` und `output`.
Die aktuelle Spezifikation aller Heat-Bestandteile und Optionen findet sich
online in der [Online-Dokumentation des Heat-Projekts](http://docs.openstack.org/developer/heat/template_guide/openstack.html)

### Heat-Version

```
heat_template_version: 2014-10-16
```

Ein Heat-Template wird eingeleitet mit der Versionsnummer der
Beschreibungssprache selbst; d.h. in dieser Zeile wird festgelegt, welches Set
von Features in den Templates verwendet werden kann und in welcher Notation.
Da die Sprache selbst sich von Release zu Release ändert, ist diese Zeile
erforderlich.

### Description

Eine Beschreibung eines Heat-Stacks ist sinnvoll, aber optional. Sinnvoll ist
sie auch deshalb, weil der Text aus dieser Sektion in der OpenStack-Datenbank
abgelegt wird und daher die hier enthaltenen Informationen abrufbar sind, wenn
der Stack gestartet ist. 

### Parameter

Mit der Parametrisierung eines Heat-Stacks wird die Wiederverwendbarkeit
erhöht: Wenn ich für ein Kundenprojekt eine sinnvolle Umgebung definiert habe,
kann ich mit leichten Anpassungen die selbe Umgebung ein weiteres Mal starten.
Hierzu dienen Parameter. Parameter werden in dieser Sektion deklariert.
Definiert werden sie entweder als Übergebene Argumente auf der Kommandozeile
(mit dem Schalter `-P <Parameter>=<Wert>`, in einem sogenannten
Environment-File, oder im Fall modular aufgebauter Heat-Stacks in dem
Eltern-Modul für das Kind-Modul als Übergabewert.  Die Deklaration sieht so
aus:

``` 
parameters:
  number_appservers
    type: string
    default: 4
```

Wie man sieht, kann ein optionaler Default angegeben werden. Wenn dieser
gesetzt ist, wird ein Parameter optional. Andersherum formuliert: Gibt es
keinen Default, wird der Parameter erzwungen, wenn man den Heat-Stack starten
möchte. Das ist beispielsweise dann sinnvoll, wenn ein Storage-Volume über
einen Parameter eingebunden werden soll, da die UID des Volumes nicht im
Vorhinein bekannt ist.

### Resources

Die Resources-Sektion ist die wichtigste in einem Template: An dieser Stelle wird festgelegt, was überhaupt gebaut werden soll. Eine Resource kann ein beliebiges Objekt in OpenStack sein.  Sehr oft geht es darum, verschiedene Resourcen nicht nur anzulegen, sondern miteinandander sinnvol zu verknüpfen. Ein Beispiel: Damit eine virtuelle Maschine Netzwerk bekommt, muss sie mit einem Port (das virtuelle Äquivalent zu einer Netzwerkkarte) verknüpft werden. Diese Verknüpfung geschieht, indem innerhalb der Maschinendefinition mittels “get_resource” auf die Port-Ressource verwiesen wird:

```
resources:
  example_instance
    type: OS:Nova::Server
    properties:
      key_name: { get_param: key_name }
      image: CirrOS 0.3.2 amd64
      flavor: m1.tiny
      networks:
- port: { get_resource: example_port }

```
Wie an diesem Beispiel zu sehen ist, gibt es noch eine andere häufig genutzte Referenzierung: Mit “get_param” wir auf den Inhalt der Parameter zugegriffen, die in der Parameterdeklaration eingeführt und deren Inhalt definiert wurde.

### Output

Die Output-Sektion liefert die Möglichkeit, Parameterwerte auch nach dem
Anlegen eines Stacks abrufen zu können; sie ist völlig optional. Der Aufbau ist
einfach: einem Parameter-Namen folgt der Wert, mit dem der Parameter befüllt
werden soll. So kann beispielsweise eine Floating-IP in der Beschreibung eines
Stacks gespeichert werden:

```
outputs:
  loadbalancer_public_ip:
    description: Floating IP address of loadbalancer in public network
    value: { get_attr: [ loadbalancer_floating_ip, floating_ip_address ] }
```

## FAQ

### Ich benutze ein Ubuntu ab 15.xx und es fehlt die Default Route

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

### Mein Stack lässt sich nicht mehr löschen! Was kann ich tun?

Ein Heat-Stack folgt beim Aufbau Abhängigkeiten, die Objekte untereinander in eine bestimmte Reihenfolge bringen. Die Dependencies werden beim Rückbau eines Stack ebenfalls berücksichtigt, so dass man ein Fehlschlagen des Löschens damit umgehen kann. Ist ein Stack ohne Angabe der Dependencies aufgebaut, schlägt das Löschen gerne mit einer Fehlermeldung wie dieser Fehl:

```
Resource DELETE failed: Conflict: resources.router_subnet_connect: Router interface for subnet eaa5a91f-3f45-43cf-8714-95118aabc64c on router 487a984c-692c-4d45-80d2-2e0ee92b505d cannot be deleted, as it is required by one or more floating IPs. 
```

In diesem Fall ist die saubere Lösung, die Abhängigkeiten von Hand zu löschen. Also erst die Floating-IP, die an dem Router hängt, dann den Router selbst und danach den ganzen Stack. Oft reicht es auch, den ``heat stack-delete <stackName>`` mehrfach aufzurufen.
Die Angabe im Heat-Template ``depends_on: <myOtherResourceID>`` vermeidet diese Probleme.



