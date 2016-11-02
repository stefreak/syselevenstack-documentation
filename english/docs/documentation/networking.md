# SysEleven Stack Networking Service

[TOC]

## Overview

The SysEleven Stack Networking Service is based on the OpenStack Neutron project.

You can manage your network both via our public OpenStack API endpoints, as well as using the [Dashboard](https://dashboard.cloud.syseleven.net).

## FAQ

### How do I connect two subnets with another?

#### Prerequisites

* Access to the [Dashboard](https://dashboard.cloud.syseleven.net)
* at least 2 existing routers/networks
* different IP ranges in both networks

#### Step One: Login

Sign in at the  [Dashboard](https://dashboard.cloud.syseleven.net)

![SysEleven Login](../img/login_router.png)

#### Step Two: Create Interface on Router

To create a new interface on our router:
* Click on the left side bar, then "network" --> "router".
* Click on the first router you want to establish a connection with.
* In the new window, under "interfaces", click on "add interfaces".
* Click on the router we want to connect to and add an IP under "IP" from the related subnet. 
* Click "Submit".

Repeat the process with the other router.

![Interface Overview](../img/hostroute.png)

#### Step Three: Creating the Host Route

To create the host route:
* Click on the left side bar, then "network", then the network you want to share.
* Click on the subnet you want to connect to
* Click on "edit subnet" and go to "subnet details".
* Under "host routes" we can now set the right route.
* Specify the ip range (e.g. 10.0.0.0/24) and the ip address of the specific router interface.
* Click "Submit" to save the host route

Repeat the process with the other subnet.

![Interface Overview](../img/router-interface.png)

#### Conclusion

You connected two subnets, so they can communicate with another.

You  also set up our own host routes and got to know the networking settings in the dashboard.
