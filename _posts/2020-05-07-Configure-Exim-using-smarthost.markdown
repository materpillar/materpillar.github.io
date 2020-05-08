---
layout: post
title:  "Configure Exim to use a smarthost for sending mails"
date:   2020-04-13 22:00:00 +0200
categories: project
---

For my Wireguard and Unifi hosts, I configured unattended-upgrades to keep
the hosts automatically in a safe state. As the administrator I would like
to be informed about the state and progress of updates. Additionally, some
of the software running on the hosts (like UniFi controller) also have mailing
functions.

The hosts are located in clients subnets without a domain setup.
The mail server of the client is an internet email provider with an own
domain. I would like to use the internet email provider SMTP server and 
email adresses to send status emails to my own private/business email.
Here, we will assume the following example values:

- Email-address (domain) rented at the internet email provier: `status@example-business.org`
- SMTP server from which emails using the email-address above can be sent: `smtp.provider.org`
- my email adress where I would like to recive the emails: `my-mail@gmail.com`

I would like to use SSH for this, but I was not able to configure it properly, therefore
here I will use the STARTTLS port 587 for the SMTP server.

Note: login and password are saved in clear text! Ensure the file is only readable by root.
Do not use a mail account which you use for your private or business mails.
The mail account used here should only be used for this very purpose!

Fot this I configure exim4 in a smarthost configuration without local mail.

Note (to myself): As the hosts are not properly integrated in a domain but attached
to a FritzBox, they get integrated into the fritz.box domain when using DHCP.
For the following configuration to work, a statically assigned IP adress seems
to be mandatory, as this 'unconfigures' the domain.
The domain of the host can be checked using

```bash
# Should only show the hostname without domain
hostname -f 
# Should be empty
hostname -d
```

## Install Exim and start configuration

```bash
sudo apt install exim4
sudo dpkg-reconfigure exim4-config
```

Choose for the upcoming options:

- General type of mail configuration: `mail sent by smarthost; no local mail`
- System mail name: `example-business.org`
- IP-address to listen on for incoming SMTP connections: `127.0.0.1 ; ::1`
- Other destinations for which mail is accepted: ` `
- Visible domain for local users: `example-business.org`
- IP address or host name of the outgoing smarthost: `smtp.provider.org::587`
- Keep number of DNS-queries minimal (Dial-on-Demand)? `No`
- Split configuration file into small files? `No`

## Set login and password of internet mail provider

In `/etc/exim4/passwd.client` configure the login and password
to the SMTP server of the internet mail provider:

`<login>` in my case is `status@example-business.org` with the corresponding password.

```bash
smtp.provider.org:<login>:<password>
```

Note: login and password are saved in clear text! Ensure the file is only readable by root.
Do not use a mail account which you use for your private or business mails.
The mail account used here should only be used for this very purpose!

## Configure that mail from root and user are forwarded to an e-mail adress

In `/etc/aliases` ensure that local mails for root are forwarded to your
normal user account. Mails to your local user accounts are then forwarded
to your actual email address.

``` bash
root: <myUsername>
<myUsername>: my-mail@gmail.com
```

### Configure the correct from-addressess to the one from which the internet mail provider will send

Edit `/etc/email-adresses`:

```bash
<myUsername>: status@example-business.org
root: status@example-business.org
```

### Edit the from name for the user accounts

Replace `<your username>` and `<hostname>` with the approbriate values.

```bash
sudo chfn -f '<hostname>(<your username>)' <your username>
sudo chfn -f '<hostname>(root)' root
```


### Test:

```bash
mail -s "Testmail" my-mail@gmail.com
# End writing and do sending by Ctrl+D
```