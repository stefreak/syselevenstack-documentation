# Using CoreOS and Containers in the SysEleven Stack

## Requirements

For this tutorial you will need to use `python-openstackclient`:

```
pip install python-openstackclient
```

You will also need to set the required environment variables to use the OpenStack API, as described in [our kickstart tutorial](https://doc.syselevenstack.com/tutorials/02-kickstart/#umgebungsvariablen-in-die-shell-session-einlesen).

You also need to have a public SSH key uploaded to the SysEleven Stack.

### Starting CoreOS in the SysEleven Stack

Step one: Find the latest CoreOS image in the image list:

```
$ openstack image list | grep -i CoreOS
| c4e4ac2c-83db-4331-bf8c-3488aff057c8 | CoreOS Stable 1122.2.0                          | active |
```

You can configure CoreOS using User-Data. Here is an example showing how to start an NGINX docker container after boot:

```
#cloud-config

coreos:
  units:
    - name: "docker-apache.service"
      command: "start"
      content: |
        [Unit]
        Description=Nginx container
        Author=Me
        After=docker.service

        [Service]
        Restart=always
        ExecStartPre=-/usr/bin/docker kill nginx
        ExecStartPre=-/usr/bin/docker rm nginx
        ExecStartPre=/usr/bin/docker pull nginx
        ExecStart=/usr/bin/docker run --rm --name nginx -p 80:80 nginx
        ExecStop=/usr/bin/docker stop nginx
```

Save this to a file called [```user-data```](../img/user-data-coreos).

With this file, the following command will launch a compute instance which will start the NGINX container. It will be reachable via port 80:

```
openstack server create --image c4e4ac2c-83db-4331-bf8c-3488aff057c8 --flavor m1.micro --user-data user-data --security-group default --nic 'net-id=aec02c14-5659-4eae-a267-501a28e672b4' --key-name 'mykey' coreos-nginx
```

### Finalizing the setup and testing it

* ```ip floating create ext-net``` provides you with a floating IP you can use to make the container reachable over the internet.
* ```ip floating add <floating-IP> coreos-nginx``` assigns the floating IP to the compute instance you just started.
* ```curl http://<floating-IP>``` should now show you the welcome page of the NGINX web server.
