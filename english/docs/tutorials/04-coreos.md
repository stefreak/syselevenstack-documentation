# Using CoreOS and Containers in the SysEleven Stack

## Requirements

For this tutorial you will need to use `python-openstackclient`:

```
pip install python-openstackclient
```

You will also need to set the required environment variables to use the OpenStack API, as described in [our kickstart tutorial](https://doc.syselevenstack.com/tutorials/02-kickstart/#umgebungsvariablen-in-die-shell-session-einlesen).

You also need to have a public SSH key uploaded to the SysEleven Stack.

### Starting CoreOS in the SysEleven Stack

Step one: Upload the latest CoreOS image

```
eval $(curl https://stable.release.core-os.net/amd64-usr/current/version.txt)
wget https://stable.release.core-os.net/amd64-usr/$COREOS_VERSION/coreos_production_openstack_image.img.bz2
openstack image create --container-format bare --disk-format qcow2 --file coreos_production_openstack_image.img.bz2 "CoreOS $COREOS_VERSION"
+------------------+------------------------------------------------------+
| Field            | Value                                                |
+------------------+------------------------------------------------------+
| checksum         | 0244e1b3420b7c170e27197eed0d0025                     |
| container_format | bare                                                 |
| created_at       | 2016-11-03T16:01:11Z                                 |
| disk_format      | qcow2                                                |
| file             | /v2/images/0882e2a7-aa27-46e6-affe-8c701dc250f5/file |
| id               | 0882e2a7-aa27-46e6-affe-8c701dc250f5                 |
| min_disk         | 0                                                    |
| min_ram          | 0                                                    |
| name             | CoreOS 1185.3.0                                      |
| owner            | 2b64d96fb0434283822a1f88f00993f9                     |
| protected        | False                                                |
| schema           | /v2/schemas/image                                    |
| size             | 258548177                                            |
| status           | active                                               |
| tags             |                                                      |
| updated_at       | 2016-11-03T16:01:14Z                                 |
| virtual_size     | None                                                 |
| visibility       | private                                              |
+------------------+------------------------------------------------------+
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
openstack server create --image 0882e2a7-aa27-46e6-affe-8c701dc250f5 --flavor m1.micro --user-data user-data --security-group default --nic 'net-id=aec02c14-5659-4eae-a267-501a28e672b4' --key-name 'mykey' coreos-nginx
```

### Finalizing the setup and testing it

* ```ip floating create ext-net``` provides you with a floating IP you can use to make the container reachable over the internet.
* ```ip floating add <floating-IP> coreos-nginx``` assigns the floating IP to the compute instance you just started.
* ```curl http://<floating-IP>``` should now show you the welcome page of the NGINX web server.
