---
title: Test host name with the provided certificate
date: 2022/10/11
description: Maybe sometime when you point a domain to a server or load balancer, you want to check if the certificate is valid for that domain.
---

Let assume the domain to test is `abc.com` and the current load balancer address is `https://test-dev-public-427989674.ap-northeast-1.elb.amazonaws.com`

First you need to find the `IP` address of this load balancer, can do by using `nslookup`

```bash
dong in dong in my-blog
❯ nslookup test-dev-public-427989674.ap-northeast-1.elb.amazonaws.com
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
Name:   test-dev-public-427989674.ap-northeast-1.elb.amazonaws.com
Address: 52.197.98.215
Name:   test-dev-public-427989674.ap-northeast-1.elb.amazonaws.com
Address: 52.193.24.44

```

Then use curl to send request

```bash
❯ curl https://abc.com --resolve "abc.com:443:52.193.24.44"
curl: (60) SSL: no alternative certificate subject name matches target host name 'abc.com'
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
❯
```

Yeah, that is.
