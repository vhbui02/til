# Mailcow Best Practices

## Overview

### Mail server - a nuts and bolts definition

A mail server is the end part of the client-server architecture aimed at exchanging information in the form of e-mail (now just refered to as "mail"). Mailcow is the server portion.

A mail client can be anything, from a CLI app (`Mutt`, ...) to a full-fledged desktop app (`Gmail's Web UI`, `Mozilla Thunderbird`, `Microsoft Outlook`, ...)

- Client-side: retrieve, view mails.
- Server-side: accept, deliver/store into mailbox, forward, ... (exchange message in general). Each task is handled by a single application. This tech is chain of communication systems, there are protocols and applications implementing them. Those apps are called "agents" or "mail agents".

### The architecture of a complete mail delivery chain

```
Sender MUA
   |
  HTTP/SMTP                                     // submission
   v
Sender's MTA  --- SMTP --> Intermediate MTA
   |                            |
  SMTP (direct)                SMTP             // relay/forward/transfer
   v                            v
Receiver's MTA <-- SMTP --- Intermediate MTA
  (Postfix)
   |                                            // relay/forward/transfer
   v
Receiver's MDA
  (Dovecot)
   |
  POP3/IMAP                 r                    // retrieval
   v
Receiver's MUA

  ┏━━━━━━━━━━ Submission ━━━━━━━━━━━━━┓┏━━━━━━━━━━━━━━ Transfer/Relay ━━━━━━━━━━━━┓

                              Mailcow's stack
                           ┌─────────────────┐                    ┌┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┐
MUA ----- opport TLS ----> ┤(587)  MTA ╮ (25)├ <-- opport TLS --> ┊ 3rd-party MTA ┊
          (STARTTLS)       │           │     │     (STARTTLS)     └┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┘
    ----- implicit TLS --> ┤(465)      │     |
    ----- cleartxt ------> ┤(25)       │     |
                           |┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄|
MUA <---- opport TLS ----- ┤(143)  MDA ╯     |
          (STARTTLS)       │                 │
    <---- implicit TLS --- ┤(993)            |
                           └─────────────────┘

  ┗━━━━━━━━━━ Retrieval ━━━━━━━━━━━━━━┛
```

There are some middle fine-grainer operations between the agents, such as analyzing, filtering, bouncing, editing, ... ran by middlewares

- **Mail/Message User Agent (MUA):** the mail client.
- **Mail/Message Transfer Agent (MTA):** the "mail server" that accepts submitted emails and relays them:

  - If the MTA is responsible for the fully qualified domain name (FQDN) of the recipient, it relays the email to a Mail Delivery Agent (MDA).
  - If the MTA is not responsible for the recipient's FQDN, it relays the email to another MTA that is responsible for that domain.

- **Mail/Message Delivery Agent:** a local software that run locally on the mail server, handling the delivery email to user's mailbox (technically speaking, store the mail inside `/var/mail/user` or a database).

A MTA/MDA can handle multiple tasks.

- Dovecot is BOTH an MDA (read mails from MTA and store it on filesystem/ write into database) and IMAP/POP3 server (reads mail from filesystem/database, and serves it to the MUA)

### A comprehensive example

- Alice owns a Gmail account: `alice@gmail.com`
- Bob owns an account of Mailcow instance: `bob@mailcow.io`

**Scenario A:** Alice is using `alice@gmail.com` to send an email to `bob@mailcow.io`.

1. The email is submitted from Alice MUA to MTA `smtp.gmail.com`.
1. The email is relayed from MTA `smtp.gmail.com` to MTA `smtp.mailcow.io`.
1. The email is relayed from MTA `smtp.mailcow.io` to Mailcow MDA.
1. When Bob go into his mailbox (access his MUA), the email is fetched from Mailcow MDA and stored inside Bob's mailbox.

=> Email submission is NOT handled by MTA `smtp.mailcow.io`, it merely _passively received_ the email relayed from other MTAs.

**Scenario B:** Bob is using `bob@mailcow.io` to send an email to `alice@gmail.com`

1. The email is submitted from Bob MUA (via webmailer `SOGo`) to MTA `smtp.mailcow.io`.
2. The mail is relayed from MTA `smtp.mailcow.io` to MTA `smtp.google.com`.
3. The mail is relayed from MTA `smtp.google.com` to Google's MDA.
4. When Alice go into her mailbox (access her MUA), the email is fetched from Google's MDA and stored inside Alice's mailbox.

=> Email submission is handled by MTA `smtp.mailcow.io`, now it _actively processed_ the email to relay to other MTAs.

## Components

It's an OSS groupware/email suite:

- `ACME`: auto generate Let's Encrypt certs (**EDIT:** by August 2025, ACME is supported native inside Nginx).
- `Dovecot`: **IMAP/POP3** mail server + FTS engine `Flatcurve`.
- `Postfix`: **MTA**.
- `SOGo`: integrated **webmailer** with Cal-/Card-dav UI.
- `Netfilter`: **Fail2ban-like** integration.
- `PHP`: chosen **programming language** for most web-based mailcow applications (it's an archaic stack, though).
- `MariaDB`: FOSS **RDBMS** for storing user info, etc
- `Memcached`: **cache** for `SOGo`.
- `Nginx`: FOSS **web servers** for every components of the stack.
- `ClamAV`: **antivirus**.
- `Olefy`: Office **documents scanner** for viruses/macros/...
- `Rspamd`: **Antispam filter** with automatic learning of spam mails.
- `Unbound`: integrated **DNS server**.
- `Watchdog`: basic container status monitoring.

The Dockerized version `mailcow: dockerized` containerized them into images connecting in a bridged network.

## Features

- Authentication protocol: DKIM + SPF + DMARC + ARC
- Blacklist/Whitelist per domain + per user.
- Whitelist hosts to forward mail to Mailcow.
- Spam score management per user.
- Mailbox users can create temporary spam aliases.
- Prepend e-mail tags to Subject section.
- Move e-mails to subfolders per user
- Allow mailbox users to toggle TLS for inbound + outbound messages.
- Cache reset on SOGo per user.
- Periodically retrieve remote mailboxes.
- 2FA: Yubikey OTP, WebAuthn USB, TOTP
- Fail2ban-like integration.
- Quarantine System.
- AV scanning (e.g. macros in Office documents)
- Basic monitoring.

## Cheatsheet

### Configuration File `mailcow.conf`

```conf
# disable ClamAV to reduce RAM consumption
SKIP_CLAMD=y
# disable Flatcurve to reduce RAM consumption
SKIP_FTS=y
```

### System Resources

**Minimum:**

- CPU: >=1Ghz
- RAM: >=6GB + 1 GB swap
- Disk: 20 GB (without mail)
- Architecture: x86_64, ARM64
- OS (tested): Debian 12, Ubuntu 24.04, Alma Linux, Rocky Linux, Alpine (limited)

**RAM consumption note:**

- A pure MTA ~ 128MB RAM
- A single SOGo worker ~ 350MB RAM (def=20 workers)
- The more ActiveSync connections being used, the more RAM will be needed

Example: 15 devices with `ExchangeActiveSync` enabled and 50 concurrent IMAP connections happening at the same time should plan for 16 GB RAM.

### Incoming/Outgoing Ports

Setup `ufw` (mapping to the rules inside `INPUT` chain)

```sh
# inbound connections
# docs: https://docs.mailcow.email/getstarted/prerequisite-system/#incoming-ports
sudo ufw allow 22,465,587,143,993,110,995,4190,80,443/tcp

# outbound connections (NOTE: only if your outbound connection rule is blacklist-all, whitelist-some)
sudo ufw allow out to any port 873,80,443,
```

> [!CAUTION]
>
> If you're using Docker and `ufw`/`firewall-cmd` at the same time, chances are you will run into some problems. Docker network traffic doesn't go through the rules inside `INPUT` chain.
>
> Use `iptables-persistent` to write rules into `DOCKER` + `DOCKER-USER` custom Netfilter chain.

### Force `STARTTLS` on port 25

Traditionally, all server-to-server (MTA-to-MTA) SMTP connections takes place over port 25. Now, SMTP protocol are updated with Opportunistic TLS: for outbound connections from your MTA server to Gmail's SMTP servers on their port 25, you can advertise `STARTTLS` to them. If they're okay, then a secure connection took place. For inbound connections to your MTA server on port 25, DO NOT ALLOW non-TLS SMTP connections, force the other MTAs to establish secure connections to you.

Port 25 itself isn't insecure, how you use it is insecure. DO NOT BLOCK port 25 since your MTA server need to trigger `STARTTLS` command to establish a secure connection with other MTAs in the first place, and vice-versa.

For client-to-server (MUA-to-MTA) communication, ISP usually blocks outgoing connections on port 25, especially at home network/consumer-level. Your MTA server must only allow secure connections from MUA via port 465 (implicit TLS), or port 587 for mail submission (exclusive operation only happened between MUA and MTA).

```cf
smtpd_tls_cert_file = /etc/ssl/certs/INSERT_YOUR_OWN_CERT.pem
smtpd_tls_key_file = /etc/ssl/private/INSERT_YOUR_OWN_KEY.pem
smtpd_tls_security_level = may      # ensure STARTTLS is advertised, allow fallback
smtpd_tls_security_level = encrypt  # requires ALL inbound SMTP connections to use TLS.
                                    # for MTAs such as Gmail, this won't be a problem
smtpd_tls_auth_only = yes
```

Restart Postfix and test advertisement:

```sh
sudo systemctl restart postfix
openssl s_client -connect localhost:25 -starttls smtp

openssl s_client -connect smtp.gmail.com:25 -starttls smtp
openssl s_client -connect smtp.gmail.com:465
openssl s_client -connect smtp.gmail.com:587 -starttls smtp

# SOF: https://stackoverflow.com/a/13772865/9122512
EHLO smtp.gmail.com
AUTH LOGIN
# => 334 VXNlcm5hbWU6 # "username" in Base64
# insert your username in Base64
# => 334 UGFzc3dvcmQ6 # "password" in Base64
# insert your password in Base64
# => 235 2.7.0 Accepted
```

### Self-signed certificates and private keys

These files made of a self-signed ceritificate that gets installed on your system when you first install `ssl-cert` package.

- `/etc/ssl/certs/ssl-cert-snakeoil.pem`
- `/etc/ssl/private/ssl-cert-snakeoil.key`

Since cert won't get regenerated after the package gets updated so some certs can be quite old (I read a dude have a cert from 2008, to the point that his latest PostgreSQL version in 2019 can't even read it).

```sh
# regenerate the self-signed cert file
make-ssl-cert generate-default-snakeoil --force-overwrite
```

It's not signed by a Certificate Authority, which means it's NOT a trusted certificate, by reading the subject content of a cert, you can see that the Common Name (the FQDN which it's valid for, the cert ensures the validity of the domain), is `localhost`

```sh
openssl x509 -in /etc/ssl/certs/ssl-cert-snakeoil.pem -noout -subject
# subject=CN = localhost
```

If you have a domain, get a cert for it. Cert is essential when real user connect to your mail service and they may get a certificate warning. The MUA compare the configured domain for incoming and outgoing mail, with the use of Common Name.

### Postfix `data/conf/postfix/extra.cf`

Postfix has two main configuration files:

- `main.cf`: global configuration settings for all SMTP processes in Postfix.
- `master.cf`: override specific settings from `main.cf`. E.g. set a global message size limit in `main.cf`, but override it for a particular service in `master.cf`

```conf
# define submission service for mail submission from clients
# use cases: sending email verification and password reset OTP
submission inet n - y - - smtpd
  -o syslog_name=postfix/submission
  -o smtpd_tls_security_level=encrypt
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_tls_auth_only=yes
  -o smtpd_reject_unlisted_recipient=no
  -o smtpd_recipient_restrictions=permit_sasl_authenticated,reject
  -o milter_macro_daemon_name=ORIGINATING
```

```conf
# Enable SMTP auth for incoming connections
smtpd_sasl_auth_enable = yes
# Disable access without TLS
smtpd_tls_auth_only = yes
# Use Dovecot as the SASL authentication backend
smtpd_sasl_type = dovecot
# The UNIX socket path where Postfix connects to Dovecot for authentication
smtpd_sasl_path = private/auth
# Disallow anonymous authentication, client must provide credentials
smtpd_sasl_security_options = noanonymous
# Sets the local domain (your mail server's hostname) for SASL authentication
smtpd_sasl_local_domain = $myhostname
# Allow workarounds for mail clients with non-standard SASL implementations
broken_sasl_auth_clients = yes
```

To customize these files, write a `data/conf/postfix/extra.cf` file in your `mailcow-dockerized` directory, DO NOT USE CUSTOM BIND MOUNT OR VOLUME MOUNT!

### Dovecot `data/conf/dovecot/extra.conf`

Dovecot has one main configuration file: `dovecot.conf`

```conf
# use Dovecot authentication as SASL authentication provider
service auth {
  unix_listener /var/spool/postfix/private-auth {
    mode = 0666
    user = vmail/mail/...
    group = vmail/mail/...
  }
}
```

To customize it, write a `data/conf/dovecot/extra.conf` file in your `mailcow-dockerized` directory, DO NOT USE CUSTOM BIND MOUNT OR VOLUME MOUNT!

### DNS setup

- Mandatory records: `A, MX`
- Security: `DKIM, SPF, DMARC`
- Mail clients auto-configuration: `SRV`.

### Mailcow's list of container volumes

```sh
docker volume ls
#  clamd-db-vol-1
#  crypt-vol-1 # NOTE: store key-pair for mail cryptocraphy ops, DO NOT FORGET TO BACKUP
#  mysql-socket-vol-1
#  mysql-vol-1
#  postfix-vol-1
#  redis-vol-1
#  rspamd-vol-1
#  sogo-userdata-backup-vol-1
#  sogo-web-vol-1
#  vmail-index-vol-1
#  vmail-vol-1
```

## References

- [Mailcow GitHub](https://github.com/mailcow/mailcow-dockerized)
- [Mailcow Official Docs](https://docs.mailcow.email)
