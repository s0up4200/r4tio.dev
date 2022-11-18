---
title: "Wildcard Certs"
date: 2022-11-18T20:54:35+01:00
draft: false
tags: ['ssl', 'nginx', 'letsencrypt', 'njalla']
---

```bash
#!/bin/bash

# By Brett Petc

# apt-get install -y socat
# install acme
# curl https://get.acme.sh | sh

# remove req for personal info
/root/.acme.sh/acme.sh --set-default-ca --server letsencrypt

export hostname="your.domain"
export NJALLA_Token="API"

# can swap out dns_cf with your dns provider. Make sure to export key first. 
/root/.acme.sh/acme.sh --force --issue --dns dns_njalla -d "${hostname}" -d "*.${hostname}"

mkdir -p /etc/nginx/ssl/${hostname}

# swap this out to whatever the name of the server is
/root/.acme.sh/acme.sh --force --install-cert -d "${hostname}" --key-file "/etc/nginx/ssl/${hostname}/key.pem" --fullchain-file "/etc/nginx/ssl/${hostname}/fullchain.pem" --ca-file "/etc/nginx/ssl/${hostname}/chain.pem" --reloadcmd "systemctl reload nginx"
```