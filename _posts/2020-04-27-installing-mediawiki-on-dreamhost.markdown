---
layout: post
title: "Installing mediawiki on dreamhost"
date: "2020-04-27 07:38:38 -0400"
---

There are a lot of tools available for collaboration. For collaborative knowledge sharing I really like mediawiki because:

* it allows for open collaboration but has strong editorial controls (as demonstrated in wikipedia)
* it has a giant community
* it has thoughtfully tested a lot of different approaches, and in general seems to favour simpler approaches
* it is quick to setup


# Register your domain

I currently use domains.google.com

## Point it to Dreamhost

You need to disable DNSSec since that is only offered for Google DNS servers.

Point to these name servers:

```
ns1.dreamhost.com
ns2.dreamhost.com
ns3.dreamhost.com
```

# Setup hosting at Dreamhost

https://panel.dreamhost.com/index.cgi?tree=domain.manage

Fully host this domain.

## Allow user to have SSH login

https://panel.dreamhost.com/index.cgi?tree=users.dashboard#!

Use ssh-copy-id username@domainname to copy your ssh key and allow passwordless login.

## Setup email for your wiki

I tend to setup "wiki@domain" as the admin wiki email.

Email servers for Dreamhost

* imap.dreamhost.com
* smtp.dreamhost.com

# Wait for DNS to propogate

You can test this by going to your site in your browser. It should show a "Dreamhost not setup yet" site.

Usually takes up to 30 minutes.

# Setup Lets Encrypt SSL

https://panel.dreamhost.com/index.cgi?tree=domain.secure

Wait for SSL to get setup. Up to 15 minutes. 

You will know if you visit http://yoursite and you are redirected to https://yoursite

# Install Mediawiki with the 1 click installer

Run the 1 click installer for your domain.

You will get an email when it is done. 

Copy the DB credentials (you need this for setup).

Then run the mediawiki installer by going to your website.

Download the LocalSettings.php file.

Use scp to upload it to your site.

```sh
scp LocalSettings.php user@your.site:folder/
```

## put your LocalSettings.php under version control

```sh
git init .
git add LocalSettings.php
git commit -m "After install"
```

# Update your logo

It should be 135x135.

Unsplash.com has great royalty free files.


# Be careful with file uploads

https://www.mediawiki.org/wiki/Manual:Security#Upload_security


# Setup email using SMTP

https://www.php.net/manual/en/mail.configuration.php

# Add analytics
      https://help.dreamhost.com/hc/en-us/articles/217292577-MediaWiki-overview

      setup google property
      added extension
            updated localsettings
            seeing data

# Backup

Your site
Your database

# Useful extensions to consider

LastModified

ShortURL



