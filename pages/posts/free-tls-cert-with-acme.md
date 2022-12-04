---
title: Get a free TLS certificate with acme.sh
date: 2022/12/4
description: When i'm using Azure Cloud, they do not provide a free certificate that can be used with their service, unlike AWS, so we need to find a way to get a free TLS certificate.
tag: certificate, acme.sh, zerossl
author: You
---

When i'm using Azure Cloud, they do not provide a free certificate that can be used with their service, unlike AWS, so we need to find a way to get a free TLS certificate.

Luckily when i go around the internet, i saw `acme.sh`, this is a ACME client that can use to get a free certificate from these CA.

- [ZeroSSL.com CA](https://github.com/acmesh-official/acme.sh/wiki/ZeroSSL.com-CA)(default)
- Letsencrypt.org CA
- [BuyPass.com CA](https://github.com/acmesh-official/acme.sh/wiki/BuyPass.com-CA)
- [SSL.com CA](https://github.com/acmesh-official/acme.sh/wiki/SSL.com-CA)
- [Google.com Public CA](https://github.com/acmesh-official/acme.sh/wiki/Google-Public-CA)
- [Pebble strict Mode](https://github.com/letsencrypt/pebble)
- Any other [RFC8555](https://tools.ietf.org/html/rfc8555)-compliant CA

Besind that [CertBot](https://certbot.eff.org/instructions) is also a client the implment ACME protocol and let use get a certificate from `Let's Encrypted` easily.

ACME provide several way to get a certificate, for this post i will use DNS manual mode because i dont need to create any virtual machine and just need to run this script on my Macbook and add some record into my domain setting.

#### Install acme.sh

Run this command, replace to your email address

```bash
wget -O -  https://get.acme.sh | sh -s email=helloworld@gmail.com
```

```bash
~/Desktop on ☁️  (ap-northeast-1) on ☁️
❯ wget -O -  https://get.acme.sh | sh -s email=helloworld@gmail.com
--2022-12-04 11:16:04--  https://get.acme.sh/
Resolving get.acme.sh (get.acme.sh)... 172.67.199.16, 104.21.34.62
Connecting to get.acme.sh (get.acme.sh)|172.67.199.16|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: ‘STDOUT’

-                                                [ <=>                                                                                         ]   1.01K  --.-KB/s    in 0s

2022-12-04 11:16:05 (9.84 MB/s) - written to stdout [1032]

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  214k  100  214k    0     0   298k      0 --:--:-- --:--:-- --:--:--  300k
[Sun Dec  4 11:16:05 +07 2022] Installing from online archive.
[Sun Dec  4 11:16:05 +07 2022] Downloading https://github.com/acmesh-official/acme.sh/archive/master.tar.gz
[Sun Dec  4 11:16:06 +07 2022] Extracting master.tar.gz
[Sun Dec  4 11:16:07 +07 2022] It is recommended to install socat first.
[Sun Dec  4 11:16:07 +07 2022] We use socat for standalone server if you use standalone mode.
[Sun Dec  4 11:16:07 +07 2022] If you don't use standalone mode, just ignore this warning.
[Sun Dec  4 11:16:07 +07 2022] Installing to /Users/dong.nguyen.04/.acme.sh
[Sun Dec  4 11:16:07 +07 2022] Installed to /Users/dong.nguyen.04/.acme.sh/acme.sh
[Sun Dec  4 11:16:07 +07 2022] Installing alias to '/Users/dong.nguyen.04/.zshrc'
[Sun Dec  4 11:16:07 +07 2022] OK, Close and reopen your terminal to start using acme.sh
[Sun Dec  4 11:16:07 +07 2022] Installing cron job
crontab: no crontab for dong.nguyen.04
crontab: no crontab for dong.nguyen.04
[Sun Dec  4 11:16:11 +07 2022] Good, bash is found, so change the shebang to use bash as preferred.
[Sun Dec  4 11:16:13 +07 2022] OK
[Sun Dec  4 11:16:13 +07 2022] Install success!

~/Desktop on ☁️  (ap-northeast-1) on ☁️   took 9s
❯
```

#### Issue an certificate

I owned the domain `dongnguyen.link` , and will try to create a certificate for `test.dongnguyen.link`.

Let run the command bellow

`acme.sh --issue --dns -d test.dongnguyen.link  --yes-I-know-dns-manual-mode-enough-go-ahead-please`

```bash
~ via  v18.0.0 on ☁️  (ap-northeast-1) on ☁️
❯ acme.sh --issue --dns -d test.dongnguyen.link  --yes-I-know-dns-manual-mode-enough-go-ahead-please
[Sun Dec  4 11:32:09 +07 2022] Using CA: https://acme.zerossl.com/v2/DV90
[Sun Dec  4 11:32:09 +07 2022] Create account key ok.
[Sun Dec  4 11:32:09 +07 2022] No EAB credentials found for ZeroSSL, let's get one
[Sun Dec  4 11:32:12 +07 2022] Registering account: https://acme.zerossl.com/v2/DV90
[Sun Dec  4 11:32:19 +07 2022] Registered
[Sun Dec  4 11:32:20 +07 2022] ACCOUNT_THUMBPRINT='-lw6tUmxLeAcNsyTe73hEGaEvZDjA04ShHLA4HKmsUA'
[Sun Dec  4 11:32:20 +07 2022] Creating domain key
[Sun Dec  4 11:32:20 +07 2022] The domain key is here: /Users/dong.nguyen.04/.acme.sh/test.dongnguyen.link/test.dongnguyen.link.key
[Sun Dec  4 11:32:20 +07 2022] Single domain='test.dongnguyen.link'
[Sun Dec  4 11:32:20 +07 2022] Getting domain auth token for each domain
[Sun Dec  4 11:32:27 +07 2022] Getting webroot for domain='test.dongnguyen.link'
[Sun Dec  4 11:32:27 +07 2022] Add the following TXT record:
[Sun Dec  4 11:32:27 +07 2022] Domain: '_acme-challenge.test.dongnguyen.link'
[Sun Dec  4 11:32:27 +07 2022] TXT value: 'RPkaD43cdfiLRDNELW3ExeA7KEKdNQVeqG8K8JTq8lQ'
[Sun Dec  4 11:32:27 +07 2022] Please be aware that you prepend _acme-challenge. before your domain
[Sun Dec  4 11:32:27 +07 2022] so the resulting subdomain will be: _acme-challenge.test.dongnguyen.link
[Sun Dec  4 11:32:27 +07 2022] Please add the TXT records to the domains, and re-run with --renew.
[Sun Dec  4 11:32:27 +07 2022] Please add '--debug' or '--log' to check more details.
[Sun Dec  4 11:32:27 +07 2022] See: https://github.com/acmesh-official/acme.sh/wiki/How-to-debug-acme.sh

~ via  v18.0.0 on ☁️  (ap-northeast-1) on ☁️   took 20s
❯
```

After running this command, you will need to add some record to your dns setting, from the stdout above, i need to add a `TXT` recording with `KEY=_acme-challenge.test.dongnguyen.link` and the value will be `RPkaD43cdfiLRDNELW3ExeA7KEKdNQVeqG8K8JTq8lQ`.

I used `AWS Route53` to mange dns

![Add Route53 Record](https://dongnguyenlqtb-blog-post-md.s3.ap-southeast-1.amazonaws.com/assets/route+53+add+txt+record.png)

The `DNS` setting will take sometime to be effectived, you can use `dig` to check when it done. Same as bellow.

```bash
~ via  v18.0.0 on ☁️  (ap-northeast-1) on ☁️
❯ dig +short -t txt _acme-challenge.test.dongnguyen.link
"RPkaD43cdfiLRDNELW3ExeA7KEKdNQVeqG8K8JTq8lQ"

~ via  v18.0.0 on ☁️  (ap-northeast-1) on ☁️
❯
```

Now run command bellow

```
acme.sh --renew -d test.dongnguyen.link \
  --yes-I-know-dns-manual-mode-enough-go-ahead-please
```

After waiting for a while, you will see the result same as below.

```bash
~ via  v18.0.0 on ☁️  (ap-northeast-1) on ☁️
❯ acme.sh --renew -d test.dongnguyen.link \
  --yes-I-know-dns-manual-mode-enough-go-ahead-please
[Sun Dec  4 11:40:17 +07 2022] Renew: 'test.dongnguyen.link'
[Sun Dec  4 11:40:17 +07 2022] Renew to Le_API=https://acme.zerossl.com/v2/DV90
[Sun Dec  4 11:40:18 +07 2022] Using CA: https://acme.zerossl.com/v2/DV90
[Sun Dec  4 11:40:18 +07 2022] Single domain='test.dongnguyen.link'
[Sun Dec  4 11:40:18 +07 2022] Getting domain auth token for each domain
[Sun Dec  4 11:40:19 +07 2022] Verifying: test.dongnguyen.link
[Sun Dec  4 11:40:28 +07 2022] Processing, The CA is processing your order, please just wait. (1/30)
[Sun Dec  4 11:40:37 +07 2022] Success
[Sun Dec  4 11:40:37 +07 2022] Verify finished, start to sign.
[Sun Dec  4 11:40:37 +07 2022] Lets finalize the order.
[Sun Dec  4 11:40:37 +07 2022] Le_OrderFinalize='https://acme.zerossl.com/v2/DV90/order/0FketBRymrFIMWVISyrEaQ/finalize'
[Sun Dec  4 11:40:40 +07 2022] Order status is processing, lets sleep and retry.
[Sun Dec  4 11:40:40 +07 2022] Retry after: 15
[Sun Dec  4 11:40:57 +07 2022] Polling order status: https://acme.zerossl.com/v2/DV90/order/0FketBRymrFIMWVISyrEaQ
[Sun Dec  4 11:41:01 +07 2022] Downloading cert.
[Sun Dec  4 11:41:01 +07 2022] Le_LinkCert='https://acme.zerossl.com/v2/DV90/cert/3HbS_y1aVzwJJgi11FBgUw'
[Sun Dec  4 11:41:05 +07 2022] Cert success.
-----BEGIN CERTIFICATE-----
MIIGdjCCBF6gAwIBAgIQECqNhy9MW8qJYv2klLym4DANBgkqhkiG9w0BAQwFADBL
MQswCQYDVQQGEwJBVDEQMA4GA1UEChMHWmVyb1NTTDEqMCgGA1UEAxMhWmVyb1NT
TCBSU0EgRG9tYWluIFNlY3VyZSBTaXRlIENBMB4XDTIyMTIwNDAwMDAwMFoXDTIz
MDMwNDIzNTk1OVowHzEdMBsGA1UEAxMUdGVzdC5kb25nbmd1eWVuLmxpbmswggEi
MA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDL0TlfnDhPKmgVG98k933Cc/ys
Ra4LR160X72OMp52pPiD90H7KoZuLx6HDu0TA8j8po0+pKBv2Pwop5N/pAZOdUZ9
9NCEApS4RE41vfAsXnG6P9l9dM+DU451J1J8n9OFbhwHq2ze5NOvDr57wqjn+faN
csmN0vUqhU6LpsPSkZ0d+8e4kU+9SXEkmS/mMv2NMa7tqvM1ERqhFq5MNQEnzrWn
BsUMjxL3iRe3CDyf/Sn7Lkf/0EA2U+lhIGs82jjA2gZhVAWvS/1VA/EegZkE4w9Y
kUGbEK2Z3qV0WHuceRSO7G2WK3K8zEgVFvy9EYXDCb6D3/TYKXoFRoJqRl2PAgMB
AAGjggKAMIICfDAfBgNVHSMEGDAWgBTI2XhootkZaNU9ct5fCj7ctYaGpjAdBgNV
HQ4EFgQUayJMRIvwuamk//XhfKTg8ryn2JYwDgYDVR0PAQH/BAQDAgWgMAwGA1Ud
EwEB/wQCMAAwHQYDVR0lBBYwFAYIKwYBBQUHAwEGCCsGAQUFBwMCMEkGA1UdIARC
MEAwNAYLKwYBBAGyMQECAk4wJTAjBggrBgEFBQcCARYXaHR0cHM6Ly9zZWN0aWdv
LmNvbS9DUFMwCAYGZ4EMAQIBMIGIBggrBgEFBQcBAQR8MHowSwYIKwYBBQUHMAKG
P2h0dHA6Ly96ZXJvc3NsLmNydC5zZWN0aWdvLmNvbS9aZXJvU1NMUlNBRG9tYWlu
U2VjdXJlU2l0ZUNBLmNydDArBggrBgEFBQcwAYYfaHR0cDovL3plcm9zc2wub2Nz
cC5zZWN0aWdvLmNvbTCCAQQGCisGAQQB1nkCBAIEgfUEgfIA8AB2AK33vvp8/xDI
i509nB4+GGq0Zyldz7EMJMqFhjTr3IKKAAABhNtvtCsAAAQDAEcwRQIhANpkw4VD
SXF+DH1yNjxVXXNKO5x5gT41pX+dZLy+CelXAiBZURsu3uhPBOxsDvHNBgi3o2q1
tvev47idN/3pA0BPmAB2AHoyjFTYty22IOo44FIe6YQWcDIThU070ivBOlejUutS
AAABhNtvs+wAAAQDAEcwRQIhAM+axva4D+Cukksq9I4Yb4bHLewmROfYdGBHxa67
WD9uAiBd5B0uRHbQPqCOlHIy7k3OGCXXf2hEgpNhVZnynlUSTTAfBgNVHREEGDAW
ghR0ZXN0LmRvbmduZ3V5ZW4ubGluazANBgkqhkiG9w0BAQwFAAOCAgEAKpdqyLJv
Z1Dr5egkDSeRq1EO+P2gvzsDIarI0xMlbZJzMlRkZ6R/l4ZJcXnO4GhRj+kM0RWk
UrgKrCqe2h1UDPLwh+/XPRWsEZ+eDBzbaIlAqKItiiXvV7O2OZJhrDYP16fgz5EH
cHhpShjvp8cruwThRPtbopSntT2O+FRFtRJ3btZys8tOAlhJ6Ptnhh8i/xtFSWf6
fztoKf8NdIUjjHPwxDiqemYO71vjZqOen3gN4hFuJYJKfWoXypJ2YL1YepcJm75f
KHU5+8rmcFUY2/ftgwa0nssoRFGuLzIONucrpQXpln9V2E65U2KAbp3ri7OmLbgf
mjkydsuRuogxIihwQQSBAKqraOZNkQKIUwuU+FtUnHWAatLEcdOSbBia0FgGBZFK
i0dz/Q01r/t9WyoAypOF8sGtZLt04lwlGt2yg3LFrt4QUzWRcgaMXPPxfu32BSk1
wZaItwYYDGFgcnDdzxOsGdZ+bjk2Divm8gkt6A4nMGKpgUBrtyYL5TH3wuXdp5My
jHu+QQrRfjN6mmylMkkKu+7jEzo7SkzqU7plsP5DJowjC9GnE0gdZPtVXFC/IEu6
MNwGifJARW0wQPGBoKk+TbrDEMf9HQ01HqneFWWcKhxtlURj81HzIxnUHGQdTe7i
bR60/hJaGGq3+YzCybf+vLmOvorXNPT3J3M=
-----END CERTIFICATE-----
[Sun Dec  4 11:41:05 +07 2022] Your cert is in: /Users/dong.nguyen.04/.acme.sh/test.dongnguyen.link/test.dongnguyen.link.cer
[Sun Dec  4 11:41:05 +07 2022] Your cert key is in: /Users/dong.nguyen.04/.acme.sh/test.dongnguyen.link/test.dongnguyen.link.key
[Sun Dec  4 11:41:05 +07 2022] The intermediate CA cert is in: /Users/dong.nguyen.04/.acme.sh/test.dongnguyen.link/ca.cer
[Sun Dec  4 11:41:05 +07 2022] And the full chain certs is there: /Users/dong.nguyen.04/.acme.sh/test.dongnguyen.link/fullchain.cer

~ via  v18.0.0 on ☁️  (ap-northeast-1) on ☁️   took 48s
❯
```

All the file is located at `$HOME/.acme.sh/test.dongnguyen.link`

```
~ via  v18.0.0 on ☁️  (ap-northeast-1) on ☁️   took 48s
❯ ls -l $HOME/.acme.sh/test.dongnguyen.link
total 72
-rw-r--r--  1 dong.nguyen.04  1721590313  4399 Dec  4 11:41 ca.cer
-rw-r--r--  1 dong.nguyen.04  1721590313  6700 Dec  4 11:41 fullchain.cer
-rw-r--r--  1 dong.nguyen.04  1721590313  2301 Dec  4 11:41 test.dongnguyen.link.cer
-rw-r--r--  1 dong.nguyen.04  1721590313   560 Dec  4 11:41 test.dongnguyen.link.conf
-rw-r--r--  1 dong.nguyen.04  1721590313  1017 Dec  4 11:40 test.dongnguyen.link.csr
-rw-r--r--  1 dong.nguyen.04  1721590313   193 Dec  4 11:40 test.dongnguyen.link.csr.conf
-rw-------  1 dong.nguyen.04  1721590313  1675 Dec  4 11:32 test.dongnguyen.link.key

~ via  v18.0.0 on ☁️  (ap-northeast-1) on ☁️
❯
```

That is, thank for reading.
