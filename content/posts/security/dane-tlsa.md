---
title: "How To Install A DANE TLSA DNS Record"
date: 2022-11-19T17:31:36+01:00
draft: false
tags: ["security", "dns", "dane", "dnssec"]
---

## What is DANE TLSA?

    Encrypted communication on the Internet often uses Transport Layer Security (TLS), which depends on third parties to certify the keys used. This document improves on that situation by enabling the administrators of domain names to specify the keys used in that domain’s TLS servers. This requires matching improvements in TLS client software, but no change in TLS server software.

## How to configure DANE TLSA

1. View the certificate of your site by clicking the padlock in the browser.

2. Export the certificate. Make sure to pick Base-64 encoding. It's needed for the TLSA RR generator later.

3. Open the certificate in a text editor and add the contents to your clipboard.

4. Go to <https://www.huque.com/bin/gen_tlsa>

5. Pick DANE-EE, SPKI, SHAH-256 and paste your certificate.

6. Fill in 443 for port number, tick TCP and enter your domain.

7. Make a new entry in your DNS ZONE like this: `TLSA _443._tcp 3 1 1 HASHOUTPUT`

## Checking TLSA

You can check if it is working by running tests on [hardenize.com](https://www.hardenize.com/report/r4tio.dev/1668859478) and [internet.nl](https://internet.nl/site/r4tio.dev/1780753/)
