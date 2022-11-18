---
title: "How to delete credentials from a Yubikey"
date: 2022-11-01T13:05:36+01:00
draft: false
tags: ['security', 'mfa', 'yubikey']
---

## What do we need?

```bash
sudo apt-add-repository ppa:yubico/stable
sudo apt update
sudo apt install yubikey-manager
```

## Requirements

Your YubiKey must have at least firmware 5.2.x. You can check this to open the YubiKey Manager app. If your YubiKey is lower than 5.2.x, then you can’t make use of this and you need to do a complete reset of your YubiKey.

## Procedure

- Type `ykman fido list` and enter your PIN to get a list of the credentials that are stored in your YubiKey.
- If you know the credential that you want to delete, type `ykman fido delete [credential]` and enter your PIN.
- Press Y to delete the credential.
- Type `ykman fido list` to check if the credential has been deleted.
