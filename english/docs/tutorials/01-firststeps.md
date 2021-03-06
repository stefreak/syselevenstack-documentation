# First steps

[TOC]

## Goals

* Start a compute instance
* Automated installation of a webserver running PHP, as well as a database server.

## Prerequisites

We assume, you know how to utilise ssh and ssh-keys.

## Login

Please log in at [https://dashboard.cloud.syseleven.net](https://dashboard.cloud.syseleven.net)
with your username and password (API credentials), provided from us.

![SysEleven Login](../img/login.png)

## Importing your ssh-key

In order to import your ssh-key using the dashboard, we go to "Compute" and "Access And Security". From there we can head to "Key Pairs" and finaly "Import Key Pair".

We have to give the key pair a name and copy/paste the public part of the key pair into the interface. Once submitted we can use the key pair with this name in the API and Templates.

![import ssh key](../img/sshkeys.png)

## Starting the compute instance

We have already created a template for our Orchestration Service for you. Feel free to check it out at [lamp.yaml](https://raw.githubusercontent.com/syseleven/heattemplates-examples/master/lampServer/lamp.yaml).

At Orchestration --> Stacks --> Launch Stack we can now use the template we were just looking at to start our server.

If we set the "Template Source" to "URL", we can now fill in the URL of the the above template into "Template URL" and go to "Next". We will set a name for the Stack at "Stackname", like *lampserver* and "key_name" should be the name of the key pair we uploaded initially. "Password" can be set to any value you like.
  
![Launch Stack](../img/launch.png)

## Login to the compute instance

* To log in to our compute instance, we have to gather its IP address. We can find our new instance at Compute --> Instances. "Floating IP" represents the public IP of our new compute instance. ```ssh syseleven@<floating-ip>``` should allow us to connect to the instance.  

![ssh login](../img/loginterminal.png)

* In the background, the webserver, database server and a current PHP version is being installed.
You can check the progress via ```tail -f /var/log/cloud-init-output.log```

Furthermore, you can now simply upload your PHP application to /var/www/html and run it.
