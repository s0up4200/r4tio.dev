---
title: "How to authenticate SSH servers with SSHFP"
date: 2022-11-20
draft: false
tags: ["security", "linux", "ssh"]
---

## Securely connecting to your server

One of SSH's key features is the use of keys to identify the server you are connecting to, meaning that you can be certain you are connecting to the correct server and not another server pretending to be that one. Unfortunately, users who connect to many servers can be accustomed to seeing the warning message that a server's key is unknown.

SSHFP presents a potential solution to this problem. This works by using the DNS system to publicly advertise the key fingerprint of the server, allowing any user to verify that the server they are connecting to is the correct one by comparing the key fingerprint presented by the server with the one held in DNS. Once enabled, you'll only ever see a warning about a server's SSH identity key if there's no SSHFP record for it in DNS, or if there's a problem with the key.

SSHFP presents a potential solution to this problem. This works by using the DNS system to publicly advertise the key fingerprint of the server, allowing any user to verify that the server they are connecting to is the correct one by comparing the key fingerprint presented by the server with the one held in DNS. Once enabled, you'll only ever see a warning about a server's SSH identity key if there's no SSHFP record for it in DNS, or if there's a problem with the key.

## Set Up SSHFP Using DNSSEC

First, you need to configure your domains with SSHFP records that allow the SSH keys on the servers to be identified. Fortunately, the ssh-keygen tool makes this a very simple process. On the server you wish to identify you simply need to run:

```text
ssh-keygen -r $(hostname)
```

Provided you have your server's hostname set up correctly with the fully qualified domain name, you should see an output like this:

`mydomain.com IN SSHFP 3 1 b4049d10454f2f5cb74688b5be2843ee08a2add5`

`mydomain.com IN SSHFP 4 1 f52df2cea30e6a435f3fdb4be70c72acdb73bfe0`

The structure can be read like this:

`⟨Name⟩ [⟨TTL⟩] [⟨Class⟩] SSHFP ⟨Algorithm⟩ ⟨Fingerprint Type⟩ ⟨Fingerprint⟩`

### Public key algorithms

There are five different algorithms defined in SSHFP. Each algorithm is represented by an integer. These are:

* 1 - RSA
* 2 - DSA
* 3 - ECDSA
* 4 - Ed25519
* 5 - Ed448

### Fingerprint type

Two fingerprint types are defined in SSHFP. Each fingerprint type is represented by an integer. These are:

* 1 - SHA-1
* 2 - SHA-256

## Add to your DNS zone

| TYPE | HOSTNAME | ALGORITHM | TYPE | TARGET | TTL |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `SSHFP`| @ | `ed25519` | `SHA-256` | `f52df2cea30e6a435f3fdb4be70c72acdb73bfe0` | 3h
| `SSHFP` | @ | `ecdsa` | `SHAH-256` | `b4049d10454f2f5cb74688b5be2843ee08a2add5` | 3h

Once this is saved and your name server is serving these records, you can check for them by digging:

`dig SSHFP mydomain.com`

The last step is to configure SSH on your local machine to accept SSHFP records for server identification. This can be done by editing the global SSH configuration on your machine, or your user's ssh config.

Add the following line to the relevant hosts or globally:

`VerifyHostKeyDNS yes`

This means that it will accept a matching DNS entry for a server's SSH identity key every time. It's also valid to set this value to “ask” instead of “yes”. If you save and exit that file then the next time you attempt to connect to a server, SSH will attempt to identify it using SSHFP if possible before falling back to asking you to manually accept the host key.

Note that if you want other people who are connecting to your server to benefit from SSHFP, you'll need to inform them to make this change to their local SSH configuration.

Sources: [Medium](https://medium.com/20ms/how-to-authenticate-ssh-servers-with-sshfp-4f9e8023995a) and [Stackexchange](https://unix.stackexchange.com/questions/121880/how-do-i-generate-sshfp-records)
