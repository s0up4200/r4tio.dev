---
title: "Adding Docker Applications to Swizzin Installs"
date: 2022-11-19T01:54:35+01:00
draft: false
author: brettpetch
tags: ["swizzin", "docker"]
editPost:
  URL: "https://brettpetch.ca/2022/11/08/adding-docker-applications-to-swizzin-installs/"
  Text: "Source" # edit text
  appendFilePath: false # to append file path to Edit link
---

After the recent r/seedboxes [Guides Post](https://redd.it/t8f9nf), there was some interest in knowing how to integrate something dockerized into the Swizzin panel. First we’re going to start at the highest level, installing Docker. Then we’ll move over to creating a compose, creating a systemd for the compose, how to put it behind nginx, and finally how to add it into the Swizzin panel. In this guide, we’ll be using Hotio’s qbittorrent image as an example by popular request.

**Please do not flood the Swizzin Discord asking for support on this. It is not supported.**

First, we’re going to install docker using the convenience script and install docker compose (v2). This guide is written exclusively for Debian and Ubuntu systems. It may not be exactly the same setup for other OS.

## Install docker and docker compose

```bash
# Elevate to root
sudo su -

# Grab docker convenience script
curl -fsSL https://get.docker.com -o get-docker.sh
sh ./get-docker.sh

# Add your user to the docker group (optional)
usermod -a -G docker yourusername

exit # this de-elevates us from root.
```

Now that we have Docker and Docker Compose setup and ready to go, we can effectively create our “project” folder. Docker uses “projects” as a way of managing multiple environments. Each “project” has its own network, which pretty much makes it sovereign from the rest of your system.

A note on Docker networks: these networks are unable to reach the arrs by default. You need to do some additional configuration, like changing bind addresses to `*` or `0.0.0.0`.

>You must secure applications using their own auth if you change the bind address!
These applications will be exposed on every interface on their port unless configured otherwise. You are responsible for ensuring that it is secured.

## Create docker compose file

I’m going to use `$HOME/.docker` as my location for dealing with it, but you may choose to use something like `/opt/` instead.

```bash
mkdir -p $HOME/.docker
```

Now, we’re going to create our `docker-compose.yml` file inside that directory. I’ll be using nano, as that’s the editor that I’m most comfortable in. You’ll need a vpn config that allows you to open ports on standby, and you’ll also need to get the id of your user, you can do this by doing id as your user.

```bash
nano $HOME/.docker/docker-compose.yml
```

Documentation: <https://hotio.dev/containers/qbittorrent/>

```yaml title="docker-compose.yml"
version: "3.8"

# Docs: https://hotio.dev/containers/qbittorrent/
services:
  qbit: # this is what you can call the service itself using
    image: hotio/qbittorrent
    container_name: qbittorrent_vpn
    ports:
      - 8118:8118
      - 127.0.0.1:8080:8080 # set like this to only expose on localhost
    volumes:
      - /home/yourusername/.docker/qbittorrent:/config
      - /home/yourusername/:/home/yourusername
    environment:
      - PUID=1000 # run `id username` as your user to get the correct value
      - PGID=1000 # run `id username` as your user to get the correct value
      - UMASK=002
      - TZ=Etc/UTC # Change this to your timezone
      - VPN_ENABLED=true
      - VPN_LAN_NETWORK
      - VPN_CONF=wg0
      - VPN_ADDITIONAL_PORTS
      - VPN_IP_CHECK_DELAY=5
      - VPN_IP_CHECK_EXIT=true # kills qbit if ip leak
      - PRIVOXY_ENABLED=false
    cap_add:
      - NET_ADMIN # required for kernel wg support. To ensure support, sudo apt-get update && sudo apt-get install linux-headers-$(uname -r)
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
      - net.ipv6.conf.all.disable_ipv6=0
```

## Systemd service

Now that our compose is created, we can go create our systemd service.

```bash
sudo nano /etc/systemd/system/qbittorrentvpn@.service
````

```systemd title="qbittorrentvpn@.service"
[Unit]
Description=qbittorrentvpn for %i
After=docker.service
Requires=docker.service
StartLimitBurst=3
StartLimitIntervalSec=30
 
[Service]
RestartSec=5
TimeoutStartSec=0
Restart=on-failure

# Compose up
ExecStart=/usr/bin/docker compose -f /home/%i/.docker/docker-compose.yml up
ExecStop=/usr/bin/docker compose -f /home/%i/.docker/docker-compose.yml stop

[Install]
WantedBy=multi-user.target
```

To control it, you can do:

```bash
sudo systemctl enable --now qbittorrentvpn@yourusername
```

## Nginx

```bash
sudo nano /etc/nginx/apps/qbittorrent-vpn.conf
````

```nginx title="/etc/nginx/apps/qbittorrent-vpn.conf"
location /qbittorrentvpn {
    return 301 /qbittorrentvpn/;
}

location /qbittorrentvpn/ {
    proxy_pass              http://127.0.0.1:8080;
    proxy_http_version      1.1;
    proxy_set_header        X-Forwarded-Host        $http_host;
    http2_push_preload on; # Enable http2 push
    auth_basic "What's the password?";
    auth_basic_user_file /etc/htpasswd.d/htpasswd.yourusername; # change yourusername with the username of your masteruser
    rewrite ^/qbittorrentvpn/(.*) /$1 break;
    proxy_cookie_path / "/qbittorrentvpn/; Secure";
}
```

Test the configuration:

```bash
sudo nginx -t
```

> nginx: the configuration file /etc/nginx/nginx.conf syntax is ok  
nginx: configuration file /etc/nginx/nginx.conf test is successful

```bash
sudo systemctl reload nginx
````

## Panel

```bash
sudo nano /opt/swizzin/core/custom/profiles.py
```

```python
### DO NOT EDIT ANYTHING ABOVE THIS LINE ###
class qbittorrentvpn_meta:
    name = "qbittorrentvpn" # name of app
    pretty_name = "qBittorrentvpn" # Name for sidebar
    baseurl = "/qbittorrentvpn" # baseurl
    systemd = "qbittorrentvpn"
    img = "qbittorrent"
    runas = "root"
```

This tells panel that an app has been installed:

```bash
sudo touch /install/.qbittorrentvpn
sudo systemctl restart panel
```

You should now have a working install that you can control with panel.

For a rootless setup, maybe checkout userdocs post from around a year ago or so. Or see here: <https://docs.docker.com/engine/security/rootless/>
