---
title: "Xbackbone"
date: 2022-11-19T01:54:35+01:00
draft: false
author: beeboo
tags: ["swizzin", "nginx", "xbackbone"]
---

## Installer script for xbackbone

This script is meant to be run on swizzin.
Read more about xbackbone here: <https://xbackbone.app>

```bash
#!/bin/bash
# XBackBone Installer for Swizzin
# By Brett
# shellcheck source=sources/globals.sh
. /etc/swizzin/sources/globals.sh

if [[ ! -f /install/.nginx.lock ]]; then
    echo_error "This script is meant to be used in conjunction with nginx and it has not been installed. Please install nginx first and restart this installer."
    exit 1
fi

function _LE() {
    if ask "Are you installing this knowing that you need a SEPERATE sub.domain or unique domain?"; then
        :
    else
        :
        exit 1
    fi

    if ask "We're going to sit here until you've setup your domain CNAME, or A/AAA record for the domain. Answer yes when done."; then
        :
    else
        :
        exit 1
    fi

    echo_query "Enter domain name you'd like to use for XBackBone to run on."
    read -e domain

    export LE_HOSTNAME="${domain}"
    export LE_DEFAULTCONF=no

    /etc/swizzin/scripts/install/letsencrypt.sh
}

function _install() {
    apt_install php-sqlite3 php-mysql php-gd php-json php-intl php-fileinfo php-zip sqlite3 libsqlite3-0

    # Get latest release and download to /tmp/

    XBACKBONE_RELEASE=$(curl -sX GET "https://api.github.com/repos/SergiX44/XBackBone/releases/latest" | awk '/tag_name/{print $4;exit}' FS='[""]')
    curl -sL "https://github.com/SergiX44/XBackBone/releases/download/${XBACKBONE_RELEASE}/release-v${XBACKBONE_RELEASE}.zip" -o /tmp/xbackbone.zip || {
        echo "unable to download xbackbone release from GitHub"
        exit 1
    }
    unzip -q -o /tmp/xbackbone.zip -d /srv/xbackbone/
    rm /tmp/xbackbone.zip
    cp /srv/xbackbone/config.example.php /srv/xbackbone/config.php
    rm /srv/xbackbone/config.example.php
    sed -i "s|localhost|${domain}|g" /srv/xbackbone/config.php

    php /srv/xbackbone/bin/migrate --install >> $log 2>&1
    rm -rf /srv/xbackbone/install
    # own it all to user
    chown -R www-data:www-data /srv/xbackbone
    echo_success "Xbackbone installed"
    touch /install/.xbackbone.lock
}
function _nginx() {

    #shellcheck source=sources/functions/php
    . /etc/swizzin/sources/functions/php
    phpversion=$(php_service_version)
    sock="php${phpversion}-fpm"

    cat > /etc/nginx/sites-enabled/xbackbone.conf << EOF
server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2;
  server_name ${domain};
  ssl_certificate /etc/nginx/ssl/${domain}/fullchain.pem;
  ssl_certificate_key /etc/nginx/ssl/${domain}/key.pem;
  include snippets/ssl-params.conf;
  client_max_body_size 2G;
  server_tokens off;
  root /srv/xbackbone/;
  index index.html index.htm index.php;
  error_page 404 /index.php;
  location ~ /\.ht {
    deny all;
  }
   location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }
    location ~ \.php$ {
        fastcgi_pass unix:/run/php/${sock}.sock;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        fastcgi_param PATH_INFO \$fastcgi_path_info;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME \$request_filename;
    }
    autoindex off;
    location ~ ^/(app|bin|bootstrap|resources|storage|vendor|logs) {
        return 403;
    }
}
EOF
    systemctl reload "${sock}"
    systemctl reload nginx
    echo_info "XBackBone is accessible at ${domain}"
    echo_info "Your login will be admin/admin. Please change this."
    echo_info "XBackBone documentation can be found at https://xbackbone.app/"
}

function _remove() {
    rm -rf /srv/xbackbone
    rm -rf /etc/nginx/sites-enabled/xbackbone.conf
    systemctl reload nginx
}

echo "Welcome to XBackBone installer..."
echo ""
echo "What do you like to do?"
echo ""
echo "install = Install XBackBone"
echo "uninstall = Completely removes XBackBone"
echo "exit = Exits Installer"
while true; do
    read -r -p "Enter it here: " choice
    case $choice in
        "install")
            _LE
            _install
            _nginx
            break
            ;;
        "uninstall")
            _remove
            break
            ;;
        "exit")
            break
            ;;
        *)
            echo "Unknown Option."
            ;;
    esac
done
exit
```
