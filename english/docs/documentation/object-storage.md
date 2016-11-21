# SysEleven Stack Object Storage

[TOC]

## Overview

SysEleven Stack provides S3 compatible Object Storage.

It stores and retrieves arbitrary unstructured data objects via a RESTful, HTTP based API. It is highly fault tolerant with its data replication and scale-out architecture. In its implementation as a distributed eventually consistent object storage, it is not mountable like a file server.

You can create the OpenStack API to generate credentials to access the SysEleven Stack Object Storage. You can then use the S3 API with various S3 clients and/or SDKs.


### Buckets

Buckets are the logical unit SysEleven Stack Object Storage uses to stores objects. Every bucket in the SysEleven Stack as a unique name.

### Objects

Basically, SysEleven Stack Object Storage is a big key/value store. A file or file object can be assigned a file name like key, and made available under this key.

## FAQ

### Can I use S3 in the SysEleven Stack?

Yes. You need to meet the following prerequisites need:


* You need to request additional rights for your user account. To do that, please send an email to our [Cloud Support](mailto:cloudsuppor@syseleven.de).
* You need the OpenStack command line tools in a current version (`2.0` or newer).
* An S3 client, i.e., `s3cmd`d
* A change to your shell environment (you can add this to your `openrc` file):

```
export OS_INTERFACE="public"
```

When these prerequisites are met, you can generate and display S3 credentials:

```
openstack ec2 credentials create
openstack ec2 credentials list
```

Now you can create an S3 configuration which should look like this:

```
syseleven@kickstart:~$ cat .s3cfg
[default]
access_key = REPLACE ME
secret_key = REPLACE ME
host_base = s3.cloud.syseleven.net
host_bucket = %(bucket).s3.cloud.syseleven.net
use_https = True
check_ssl_certificate = True
check_ssl_hostname = False
```

Next, create an S3 Bucket:

```
s3cmd mb s3://BUCKET_NAME
```

Then, use it to add some file(s):

```
s3cmd put test.jpg s3://BUCKET_NAME -P
```

The command-line option `-P` means the file(s) uploaded is publicly available. Please note that `s3cmd` returns incorrect URLs, i.e.:

```
Public URL of the object is: http://BUCKET_NAME.s3.amazonaws.com/test.jpg
```

The correct URL in this case would be https://s3.cloud.syseleven.net/BUCKET_NAME/test.jpg

You can use these URLs to refer to the uploaded files as static assets in your web applications.
