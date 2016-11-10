# Minimal CoreOS Setup

[TOC]

## Goal

* You will learn how to start a minimal CoreOS instance on the SysEleven Stack.
* You will learn how to upload the current CoreOS stable image.

## Prerequisites 

* You should be able to use simple heat templates, like shown in the [first steps tutorial](01-firststeps/).
* You know the basics of using the OpenStack CLI (Environment variables are set, like shown in the [Kickstart-Tutorial](02-kickstart/).

## 1. Clone example repository

You will be working with our "[heattemplates-examples](https://github.com/syseleven/heattemplates-examples)" repository on github. Your first step is to clone it:

```
$ git clone https://github.com/syseleven/heattemplates-examples
$ cd heattemplates-examples/coreOS
```

## 2. Upload CoreOS image

After you cloned the repository, upload the current stable CoreOS image to SysEleven Stack.

```
$ ./upload_replacing_coreos_image.sh
```

This helper script downloads the [official stable CoreOS image](https://coreos.com/os/docs/latest/booting-on-openstack.html), deletes existing images on your SysEleven Stack named `private_coreos` and uploads the new image. 

## 3. Start CoreOS instances

```
$ openstack stack create -t cluster.yaml <stack name> --parameter key_name=<ssh key name> --wait
```

In this command, `key_name` references an SSH-Key that you created in the [First Steps Tutorial](01-firststeps/#importing-your-ssh-key). You can start more than one instance setting the optional parameter `number_instances`. If you do not set it, only one instance will start up.

## Conclusion

You have now started one or more CoreOS instances on the SysEleven Stack. You will be able to access each one using a public floating IP address.

Every CoreOS instance runs a docker container launching an NGINX based webserver, listening on port 80. You can check that using the following command:

`curl <ip-address>` 

For every CoreOS instance, you should see the default NGINX welcome page.

This setup is still missing a couple pieces to provide load balancing or high availability. Check out the links in the next section for some advanced setups you can build based on this tutorial.


## Links/Examples

* [CoreOS cluster discovery](https://coreos.com/os/docs/latest/cluster-discovery.html)
* [Running a High Availability Docker Swarm](http://tech.paulcz.net/2016/01/running-ha-docker-swarm/)
