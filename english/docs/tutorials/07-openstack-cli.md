# Openstack CLI

[TOC]

## Overview

This tutorial will help you to install the OpenStack CLI (Command Line Interface).
With the OpenStack CLI you can adminstrate your Stacks.

## Prerequisites

* PC/MAC
* admin rights
* basic computer skills

We expect that you havn't installed any of the needed tools..
If you already installed any of the tools, please skip that step.

### Installation Mac

Optional you can install an alternative Terminal like [iTerm2](https://www.iterm2.com/).

Open the Terminal or if you installed iTerm2, open iTerm2.
To install the needed packages, we will use [pip](https://en.wikipedia.org/wiki/Pip_(package_manager)) as package manager.

You can install pip with this command:
```
easy_install pip
```

After you have installed PIP, it is only a small command to install the OpenStack Client.
```
pip install python-openstackclient
```

### Installation Windows

If you want to install the OpenStack Client on windows, we will need first [Python 2.7](https://www.python.org/downloads/release/python-2712/)
After you have installed Python, open your Command Promt and ensure that you're in the C:\Python27\Scripts directory.

Now use the easy_install command to install [pip](https://en.wikipedia.org/wiki/Pip_(package_manager)) as package manager:
```
C:\Python27\Scripts>easy_install pip
```

After the installation finished, install now the OpenStack CLI
```
C:\Python27\Scripts>pip install python-openstackclient
```

### Installation Red Hat Enterprise Linux, CentOS or Fedora

To install the OpenStack Client yo will need to install pip first [pip](https://en.wikipedia.org/wiki/Pip_(package_manager))
```
yum install python-devel python-pip
```

Now the only thing left, is the installation of the OpenStack Client:
```
pip install python-openstackclient
```

### Optional Plugins

There is a possibilit to install additional Plugins.
If you need to install some plugins, replace PLUGINS with one of the list below
```
pip install python-PLUGINclient
```

Plugin list:

* barbican - Key Manager Service API
* ceilometer - Telemetry API
* cinder - Block Storage API and extensions
* cloudkitty - Rating service API
* designate - DNS service API
* fuel - Deployment service API
* glance - Image service API
* gnocchi - Telemetry API v3
* heat - Orchestration API
* magnum - Container Infrastructure Management service API
* manila - Shared file systems API
* mistral - Workflow service API
* monasca - Monitoring API
* murano - Application catalog API
* neutron - Networking API
* nova - Compute API and extensions
* sahara - Data Processing API
* senlin - Clustering service API
* swift - Object Storage API
* trove - Database service API

### Conclusion

We have now installed the OpenStack Client and we can use it like in this Example [SysEleven Stack API - Beginner Tutorial](https://doc.syselevenstack.com/en/tutorials/02-kickstart)
If needed you can list all commands:
```
openstack --help
```