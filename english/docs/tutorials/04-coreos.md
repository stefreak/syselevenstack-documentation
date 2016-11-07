# Minimal CoreOS Setup

[TOC]

## Goal

* Wir werden eine minimale CoreOS Instanz im SysEleven Stack starten.
* Außerdem laden wir das aktuellste CoreOS image hoch

## Prerequisites 

* Der Umgang mit einfachen Heat-Templates, wie in den ersten Schritten gezeigt, wird vorausgesetzt.
* Grundlagen zur Bedienung des OpenStack CLI (Umgebungsvariablen gesetzt, wie im [Kickstart-Tutorial](02-kickstart/) beschrieben.

## 1. Clone example repository

We will be working with the files in our "[heattemplates-examples](https://github.com/syseleven/heattemplates-examples)" repository on github. Please clone it.

```
$ git clone https://github.com/syseleven/heattemplates-examples
$ cd heattemplates-examples/coreOS
```

## 2. Upload CoreOS image

Upload the CoreOS image to SysEleven Stack.

```
$ ./upload_replacing_coreos_image.sh
```

This helper script downloads the [official stable CoreOS image](https://coreos.com/os/docs/latest/booting-on-openstack.html), deletes existing images on your SysEleven Stack named `private_coreos` and uploads the new image. 

## 3. Start CoreOS instances

```
$ openstack stack create -t cluster.yaml <stack name> --parameter key_name=<ssh key name> --wait
```

key_name references an SSH-Key that you created in the [First Steps Tutorial](01-firststeps/#importing-your-ssh-key).

Using the optional parameter `number_instances` (default: 1) you can start multiple instances.

## Conclusion

Now one or more CoreOS instances should run on the SysEleven Stack. Each one is accessible using a public FloatingIP. On port 80 an Nginx, running inside of a docker container, should respond to HTTP requests.

Command for checking this:
`curl <ip-address>` (Should show the Nginx welcome page)

For a High Availability Cluster we are still missing some stuff. More advanced setups are explained further in the links below.

## Links/Examples

* [CoreOS cluster discovery](https://coreos.com/os/docs/latest/cluster-discovery.html)
* [Running a High Availability Docker Swarm](http://tech.paulcz.net/2016/01/running-ha-docker-swarm/)