---
title: "How to protect domains that do not send emails"
date: 2022-11-21
draft: false
tags: ["security", "email", "dns", "dmarc", "dkim", "spd"]
author: Cloudflare
editPost:
  URL: "https://www.cloudflare.com/learning/dns/dns-records/protect-domains-without-email/"
  Text: "Source" # edit text
  appendFilePath: false # to append file path to Edit link
---

Domains that do not send emails can still be used in email [spoofing](https://www.cloudflare.com/learning/ssl/what-is-domain-spoofing/) or [phishing attacks](https://www.cloudflare.com/learning/access-management/phishing-attack/), but there are specific types of [DNS text (TXT)](https://www.cloudflare.com/learning/dns/dns-records/dns-txt-record/) records that can be used to stifle attackers. Each of these records sets rules for how unauthorized emails should be treated by mail servers, making it harder for attackers to exploit these domains.

A DNS TXT record allows domain administrators to enter text into the [Domain Name System (DNS)](https://www.cloudflare.com/learning/dns/what-is-dns/). DNS TXT records are used for processes like email authentication because they can store important information that servers can use to confirm whether or not a domain has authorized an email sender to send messages on its behalf.

Examples of domains that do not send emails include domains purchased to protect a brand name or for a future business. Defunct, legacy domains also have no reason to send emails and could benefit from these types of records.

## What are the different types of DNS TXT records used for email authentication?

There are three main types of DNS TXT records used for email authentication. Each of them differs slightly in how they work:

- [Sender Policy Framework (SPF)](https://www.cloudflare.com/learning/dns/dns-records/dns-spf-record/) records list all of the IP addresses and domain names that are authorized to send emails on behalf of the domain.

- [DomainKeys Identified Mail (DKIM)](https://www.cloudflare.com/learning/dns/dns-records/dns-dkim-record/) records provide a digital signature to authenticate whether or not the sender actually authorized the email. This digital signature also helps prevent [on-path attacks](https://www.cloudflare.com/learning/security/threats/on-path-attack/), in which attackers intercept communications and alter messages for nefarious purposes.

- [Domain-based Message Authentication, Reporting and Conformance (DMARC)](https://www.cloudflare.com/learning/dns/dns-records/dns-dmarc-record/) records hold a domain’s DMARC policy. DMARC is a system for authenticating emails by checking a domain’s SPF and DKIM records. The DMARC policy states whether emails that fail SPF or DKIM checks should be marked as spam, blocked, or allowed to go through.

## What do these DNS records look like?

Because these DNS records all function slightly differently, each of their components are unique.

### SPF

SPF records can be formatted to protect domains against attempted phishing attacks by rejecting any emails sent from the domain. To do so, an SPF record must use the following format.

`v=spf1 -all`

*Note, SPF records are set directly on the domain itself, meaning they do not require a special subdomain.

Here is what the individual components of this record mean:

- `v=spf1` lets the server know that the record contains an SPF policy. All SPF records must begin with this component.

- The indicator `-all` tells the server what to do with non-compliant emails or any senders that are not explicitly listed in the SPF record. With this type of SPF record, no IP addresses or domains are allowed, so `-all` states that all non-compliant emails will be rejected. For this type of record, all emails are considered non-compliant because there are no accepted IP addresses or domains.

## DKIM

DKIM records protect domains by ensuring emails were actually authorized by the sender using a public key and a private key. DKIM records store the public key that the email server then uses to authenticate that the email signature was authorized by the sender. For domains that do not send emails, the DKIM record should be configured without an associated public key. Below is an example:

| TYPE | NAME | CONTENT |
| :--- | :--- | :--- |
| `TXT`| `*._domainkey.example.com` | `v=DKIM1; p=` |

- `*._domainkey.example.com` is the specialized name for the DKIM record (where “example.com” should be replaced with your domain). In this example, the asterisk (referred to as the wildcard) is being used as the selector, which is a specialized value that the email service provider generates and uses for the domain. The selector is part of the DKIM header and the email server uses it to perform the DKIM lookup in the DNS. The wildcard covers all possible values for the selector.

- `TXT` indicates the DNS record type.
v=DKIM1 sets the version number and tells the server that this record references a DKIM policy.

- The `p value` helps authenticate emails by tying a signature to its public key. In this DKIM record, the p value should be empty because there is no signature/public key to tie back to.

## DMARC

DMARC policies can also help protect domains that do not send emails by rejecting all emails that fail SPF and DKIM. In this case, all emails sent from a domain not configured to send emails would fail SPF and DKIM checks. Below is an example of how to format a policy this way:

| TYPE | NAME | CONTENT |
| :--- | :--- | :--- |
| `TXT`| `_dmarc.example.com` | `v=DMARC1;p=reject;sp=reject;adkim=s;aspf=s` |

- The name field ensures that the record is set on the subdomain called `_dmarc.example.com`, which is required for DMARC policies.

- `TXT` indicates the DNS record type.

- `v=DMARC1` tells the server that this DNS record contains a DMARC policy.

- `p=reject` indicates that email servers should reject emails that fail DKIM and SPF checks.

- `adkim=s` represents something called the alignment mode. In this case, the alignment mode is set to “s” for strict. Strict alignment mode means that the server of the email domain that contains the DMARC record must exactly match the domain in the From header of the email. If it does not, the DKIM check fails.

- `aspf=s` serves the same purpose as adkim=s, but for SPF alignment.
