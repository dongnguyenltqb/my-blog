---
title: Bastion Host grant less permission
date: 2022/10/13
description: In production environment, the access is private by default, developer usually have not permission to access to the database, search engine, or the running application. In case to debug application, running specific query on database, devops need to provide access for developer to able to access to this resource.
tag: bastion host,ec2,amazon linux 2
author: You
---

In production environment, the access is private by default, developer usually have not permission to access to the database, search engine, or the running application. In case to debug application, running specific query on database, devops need to provide access for developer to able to access to this resource.

So for each use-case, devops need to grant enough permission. Let go through some use-case.

## Use Case 1: Developer need to access to the database.

For this use-case we can provider access to the bastion host and allow user to use ssh port-forwarding only. Let’s do it. Assume you have a virtual machine which has ip `192.168.1.115` and an `openssh-server` which is listening on port `2222.`

First login to this instance.

```
ssh root@192.168.1.115 -p 2222
```

Create user credential for developer.

```
$ useradd -m -d /home/alice -s /usr/bin/bash alice
```

Configure openssh server to allow only port-forwarding

```
cd /etc/ssh
vi sshd_config
```

add this block to this file

```
Match User alice
        X11Forwarding no
        AllowTcpForwarding yes
        PermitTTY no
        ForceCommand exit
        PubkeyAuthentication yes
        PasswordAuthentication no
```

For this configuration, developer only has permission to use port-forwarding, each time use try to login to this server, the user session will be forced to exit. We only support authenticate using public key instead of password to prevent brute force attack.

Let generate a ssh keypair for this user.

```
❯ ssh-keygen -t ed25519 -f alice -C "alice-user"
Generating public/private ed25519 key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in alice
Your public key has been saved in alice.pub
The key fingerprint is:
SHA256:xeJwT6biReIIn+2GNXp8pveKbLyfP2R58zT5S99xVGM alice-user
The key's randomart image is:
+--[ED25519 256]--+
|                 |
|         .       |
|  .   o + =    E.|
|   o = * B    . o|
|    + * S ..   ..|
|     B +  + o +. |
|    o.* oo . + =.|
|     +o=...   o *|
|     .=+++o.   .+|
+----[SHA256]-----+

~/Desktop/alice-keys on ☁️  (ap-southeast-1)
❯ ll
total 8.0K
-rw------- 1 dong dong 399 Nov 13 11:44 alice
-rw-r--r-- 1 dong dong  92 Nov 13 11:44 alice.pub

~/Desktop/alice-keys on ☁️  (ap-southeast-1)
❯
```

Now add alice public key to `/home/alice/.ssh/authorized_keys`

```
[root@localhost ~]# mkdir -p /home/alice/.ssh
[root@localhost ~]# vi /home/alice/.ssh/authorized_keys
```

Now, `alice` is able to use `ssh port-forwarding` but login to this vm, if `alice` try to login and get an `PTY` session, the session will be terminated.

```
~/Desktop/alice-keys on ☁️  (ap-southeast-1)
❯ ssh -i alice alice@192.168.1.115 -p 2222
PTY allocation request failed on channel 0
Connection to 192.168.1.115 closed.
```

Now, let test it. On `alice` machine run this command

```
$ ssh -i alice -N -L 8080:google.com:80 alice@192.168.1.115 -p 2222
```

Send a request to port `8080`

```
$ curl localhost:8080
<!DOCTYPE html>
<html lang=en>
  <meta charset=utf-8>
  <meta name=viewport content="initial-scale=1, minimum-scale=1, width=device-width">
  <title>Error 404 (Not Found)!!1</title>
  <style>
    *{margin:0;padding:0}html,code{font:15px/22px arial,sans-serif}html{background:#fff;color:#222;padding:15px}body{margin:7% auto 0;max-width:390px;min-height:180px;padding:30px 0 15px}* > body{background:url(//www.google.com/images/errors/robot.png) 100% 5px no-repeat;padding-right:205px}p{margin:11px 0 22px;overflow:hidden}ins{color:#777;text-decoration:none}a img{border:0}@media screen and (max-width:772px){body{background:none;margin-top:0;max-width:none;padding-right:0}}#logo{background:url(//www.google.com/images/branding/googlelogo/1x/googlelogo_color_150x54dp.png) no-repeat;margin-left:-5px}@media only screen and (min-resolution:192dpi){#logo{background:url(//www.google.com/images/branding/googlelogo/2x/googlelogo_color_150x54dp.png) no-repeat 0% 0%/100% 100%;-moz-border-image:url(//www.google.com/images/branding/googlelogo/2x/googlelogo_color_150x54dp.png) 0}}@media only screen and (-webkit-min-device-pixel-ratio:2){#logo{background:url(//www.google.com/images/branding/googlelogo/2x/googlelogo_color_150x54dp.png) no-repeat;-webkit-background-size:100% 100%}}#logo{display:inline-block;height:54px;width:150px}
  </style>
  <a href=//www.google.com/><span id=logo aria-label=Google></span></a>
  <p><b>404.</b> <ins>That’s an error.</ins>
  <p>The requested URL <code>/</code> was not found on this server.  <ins>That’s all we know.</ins>
```

It works, you can see that we can able to forward port `8080` on local machine to port `80` on host `google.com` , you can try with `Amazon RDS.`
