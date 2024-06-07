---
layout: post
title: Haproxy and letsencrypt
date: 2022-04-10
last_modified: 2024-06-07
comments: true
---


```bash
sudo apt install haproxy
sudo snap install certbot
```


Basic haproxy and letsencrypt installation and setup.  


## Generate a cert

```bash
certbot certonly --standalone -d [yoursubdomain.of.your.site.majorsilence.com] --non-interactive --agree-tos --email [your email address] --http-01-port=8899
```

## Haproxy

/etc/haproxy/haproxy.cfg

```cfg
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1>
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

frontend letsencrypt-frontend
    bind :80
    bind :::80
    mode http
    acl destination_letsencrypt-backend00 path_beg /.well-known/acme-challenge/
    use_backend letsencrypt-backend if destination_letsencrypt-backend00

frontend my-web-app-fe
    #bind *:80
    bind *:443 ssl crt /etc/lets-ecrypt/haproxy-gen/ alpn h2,http/1.1
    #http-request redirect scheme https unless { ssl_fc }

    # detect domains
    acl destination_somedomain100 hdr_beg(host) -i subdomain1.majorsilence.com
    acl destination_somedomain200 hdr_beg(host) -i subdomain2.majorsilence.com

    # specify backends
    use_backend somedomain1-backend if destination_somedomain100
    use_backend somedomain2-backend if destination_somedomain200

backend letsencrypt-backend
    mode http
    option forwardfor
    option httplog
    server certbot 127.0.0.1:8899

backend somdomain1-backend
    balance roundrobin
    option httpchk
    server server1 ip:port check
    server server2 ip:port check
    server server3 ip:port check

backend somedomain2-backend
    balance roundrobin
    option httpchk
    server server1 ip:port check
    server server2 ip:port check
    server server3 ip:port check
```    

### backend self signed certs


```
backend somedomain2-backend
    balance roundrobin
    option httpchk GET / HTTP/1.1
    server server1 ip:port ssl verify none
    server server2 ip:port ssl verify none
    server server3 ip:port ssl verify none
```


### Test haproxy config

```bash
haproxy -c -V -f /etc/haproxy/haproxy.cfg
```


## certbot updates

How can this be setup to run after the snap certbot systemd timer runs to renew?

```bash
cd /etc/systemd/system 
ls -l *certbot*
```


/etc/cron.daily/updatecertsforhaproxy

```bash
#!/usr/bin/env bash

# Renew the certificate
certbot renew

# Haproxy requires certs concatenated
mkdir -p /etc/lets-ecrypt/haproxy-gen
bash -c "cat /etc/letsencrypt/live/subdomain1.majorsilence.com/fullchain.pem /etc/letsencrypt/live/subdomain1.majorsilence.com/privkey.pem > /etc/lets-ecrypt/haproxy-gen/subdomain1.majorsilence.com.pem"

bash -c "cat /etc/letsencrypt/live/subdomain2.majorsilence.com/fullchain.pem /etc/letsencrypt/live/subdomain2.majorsilence.com/privkey.pem > /etc/lets-ecrypt/haproxy-gen/subdomain2.majorsilence.com.pem"

systemctl reload haproxy
```

## References

* [Let's Encrypt on HAProxy](https://devops.cisel.ch/lets-encrypt-on-haproxy)
* [HAProxy with SSL and Let’s Encrypt](https://gridscale.io/en/community/tutorials/haproxy-ssl/)
* [Install Let’s Encrypt SSL on HAProxy](https://markontech.com/linux/install-lets-encrypt-ssl-on-haproxy/)
* [The Four Essential Sections of an HAProxy Configuration](https://www.haproxy.com/blog/the-four-essential-sections-of-an-haproxy-configuration/)
* [haproxy - unable to load SSL private key from PEM file](https://stackoverflow.com/questions/27947982/haproxy-unable-to-load-ssl-private-key-from-pem-file)
* [Automatically generatable Public & Private .pem file](https://community.letsencrypt.org/t/automatically-generatable-public-private-pem-file/164527)
* [How to automate certbot certificate renewal on Ubuntu 20.04](https://serverfault.com/questions/1057412/how-to-automate-certbot-certificate-renewal-on-ubuntu-20-04)
* [Let’s Encrypt with HAProxy](https://kevinbentlage.nl/blog/lets-encrypt-with-haproxy/)
