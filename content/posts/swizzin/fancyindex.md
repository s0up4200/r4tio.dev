---
title: "Fancyindex"
date: 2022-11-19T01:54:35+01:00
draft: false
author: beeboo
tags: ["swizzin", "nginx", "fancyindex"]
editPost:
  URL: "https://discord.com/channels/537625695967510539/540549627892727819/799001400163434537"
  Text: "Source" # edit text
  appendFilePath: false # to append file path to Edit link
---

1. Open up a terminal and ssh into your server. Once in, run `sudo nano /etc/nginx/apps/external.conf` and paste the contents in the codeblock below:

```nginx
# This config serves an unprotected index of files stored at /srv/external (from the softlinked path)
location /external {
    include /etc/nginx/snippets/fancyindex.conf;
}
```

2. Now, we're going to softlink the folder we want to share to /srv/external. This allows for you to share a folder from your user folder, or anywhere else on the machine to the public. `sudo ln -s /path/to/shared/directory /srv/external`
3. Test the config: `sudo nginx -t`
4. If return is OK, you can reload the nginx service with `sudo service nginx reload`.

Note: If you don't get an OK, go over your nginx configuration, make sure there aren't multiple files for each app and that no two things are being served at the same location. Do a configtest again until you find your issue, then reload the nginx service.
