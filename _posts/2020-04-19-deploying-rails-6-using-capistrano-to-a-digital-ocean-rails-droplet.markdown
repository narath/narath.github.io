---
layout: post
title: "Deploying Rails 6 Using Capistrano to a Digital Ocean Rails Droplet"
date: "2020-04-19 06:32:01 -0400"
---

Yesterday I spent a little time getting back up to speed on deploying a Rails 6 application to a [Digital Ocean Rails Droplet](https://marketplace.digitalocean.com/apps/ruby-on-rails#getting-started) using [Capistrano](https://capistranorb.com/).

# 1. Create your digital ocean droplet

You can [create your droplet using their one click installer or their api](https://marketplace.digitalocean.com/apps/ruby-on-rails#getting-started). Make sure you use your ssh key to allow your access since Capistrano depends on ssh access (and ssh agent forwarding).

This will setup

* Ubuntu
* Ruby
* Rails
* Puma
* Nginx reverse proxy
* Node.js
* Certbot (for your LetsEncrypt certificate setup and management)
* and a rails.service in /etc/systemd/system
* ufw firewall service
* a very helpful message of the day that describes passwords for postgres, sftp
* a deploy user "rails" that also vendors gems
  * so to access rails console/logs/update gems/launch puma - you need to "sudo -i -u rails"

## Where are logs stored?

### Rails logs

Once deployed with Capistrano, these will be stored in /home/rails/#{app}/shared/logs. The "rails" user has access to this. Later we will have a capistrano task to allow you to quickly tail these.

### Nginx

As root:

```sh
less /var/log/nginx/error.log
```

### Rails.service / Puma logs

As root or rails

```sh
journalctl -u rails.service
```

Add -f to follow.

```sh
journalctl -u rails.service -f
```

# 2. Setup a domain and LetsEncrypt

(Note: you don't have to do this step, you can deploy with capistrano using just an IP, but you cannot use https that way (as far as I know)).

By default, the droplet comes with an example rails application that nginx points to as a reverse proxy. This is locate in "/home/rails/example". Initially this is just running in "development" mode so by browsing to your IP you should see the "Yay! You're on Rails" view. 

## Point your domain/subdomain to your droplet ip

Add an "A" record for your domain or subdomain to point to your droplet IP. This can take up to an hour to propagate. 

If you don't change anything, then it should point to your IP but with the new [Rails DNS rebinding protection](https://prathamesh.tech/2019/09/02/dns-rebinding-attacks-protection-in-rails-6/) you would just see a "Blocked host" message (but at least you know it is pointing to the right server).

## Setup an enabled site block in Nginx for your site

Key steps:

- create nginx server block sites-available/your-site-domain-name
  - make sure the server name is set there
- make sure it is pointed to in sites-enabled
  - remember: ln -s source target_new_name
- then can ask certbot to install a certificate for you
- setup an automated renewal for your sites

### Create an Nginx server block for your site

As root

```sh
vim /etc/nginx/sites-available/your-site-domain-name
```

{% gist 56b858aa8c5dedfdea9336e79511be04 nginx_server_block %}

As you can see this was automatically modified by Certbot, and does the automatic http->https redirection.

### Enable this site

Certbot looks in sites-enabled, so make sure you create a symbolic link there.

As root

```sh
ln -s /etc/nginx/sites-available/your-site-domain-name /etc/nginx/sites-enabled/your-site-domain-name
```

At this point I remove the default site from sites-enabled (`rm /etc/nginx/sites-enabled/rails`).

### Install and run certbot

```sh
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx
```

Make sure that port 443 is open (for nginx)

```sh
sudo ufw status
sudo certbot --nginx -d your.site.name
```

### Make sure you setup a renewal

Certbot can do automated renewals. This [post](https://www.vultr.com/docs/how-to-install-and-configure-ruby-with-rbenv-rails-mariadb-nginx-ssl-and-passenger-on-ubuntu-17-04) described a nice crontab that will do the renewal for you. If your site is not due for a renewal, Certbot will not attempt renewal (which is great because you are limited to a certain number of renewals each period).

As root

```sh
sudo crontab -e 
```

```cron
30 2 * * 1 /usr/bin/certbot renew 
```

### If there is an error when you first install your certficate, you can have Certbot reinstall it for you

I found [this certbot community post](https://community.letsencrypt.org/t/certbox-could-not-automatically-find-a-matching-server-block/105265) helpful, when certbot could not initially find my server block.

```sh
certbot --reinstall -d your.site.domain-name
```

# 3. Create your Rails app and setup Capistrano to deploy it

The server is setup with Ruby 2.6.5, so use this for local development. 

## Create a Rails app with postgresql db

```sh
rails new app-name --database postgresql
```

## Setup your database config and credentials

{% gist 56b858aa8c5dedfdea9336e79511be04 database.yml %}

Then update your master.key with these credentials.

```sh
EDITOR=vim rails credentials:edit
```

{% gist 56b858aa8c5dedfdea9336e79511be04 credentials.yml %}

## Setup Capistrano

{% gist 56b858aa8c5dedfdea9336e79511be04 Gemfile %}

```sh
bundle install
bundle exec cap install

### Update your Capfile

{% gist 56b858aa8c5dedfdea9336e79511be04 Capfile %}

### Update config/deploy.rb

```

{% gist 56b858aa8c5dedfdea9336e79511be04 config_deploy.rb %}

Note: this has a basic setup for your server, but also code at the bottom that will upload your shared config files if they don't exist on the server.

### Update config/deploy/production.rb

{% gist 56b858aa8c5dedfdea9336e79511be04 config_deploy_production.rb %}

### Add Capistrano tasks

This task will let you quickly tail log files on the server.

{% gist 56b858aa8c5dedfdea9336e79511be04 lib_capistrano_logs.rake %}

This task is needed to properly restart the puma service after a deploy.

{% gist 56b858aa8c5dedfdea9336e79511be04 lib_capistrano_deploy_restart.rake %}

**Important: in order for this to work you need to make this sudo command not require a password.**

As root

```sh
sudo visudo -f /etc/sudoers.d/rails
```

```txt
rails ALL=NOPASSWD: /bin/systemctl restart rails.service
```

This was a helpful [reference](https://teamtreehouse.com/library/restarting-unicorn). Note: you are only allowing that single command to be run by that user as sudo without a password, so it is not making all sudo commands not require a password.

# 4. Deploy

## Make sure your ssh agent is setup

On your mac, ssh agent should already be setup. Check to see that the agent has your identity loaded.

```sh
ssh-add -l
```

If not, then just enter `ssh-add` and your default identity should be loaded.

## Do a partial deploy to be able to setup the database

You should now be ready to deploy. There is still a little work since your database has not been setup. 

```sh
cap production deploy --trace
```

This will break because the DB is not setup, but the code is there now with the credentials to do the setup.

As rails

```sh
cd app-name/releases
```

Go into the latest release folder

```sh
RAILS_ENV=production bundle exec rails db:setup
```

This should setup the db. 

## Do a complete deployment

Now rerun your deployment and it should work.

```sh
cap production deploy --trace
```

# 5. Update and restart your services

You need to update rails.service to use production and point to your app. As root:

```sh
sudo vim /etc/systemd/system/rails.service
```

Make sure the WorkingDirectory and ExecStart.

```
WorkingDirectory=/home/rails/appname/current
ExecStart=/bin/bash -lc 'RAILS_ENV=production bundle exec puma'
```

Reload and restart the services.

```sh
systemctl daemon-reload
systemctl restart rails.service
journalctl -u rails.service -f
```

# Appendix/FAQ

## How to access Postgres as the postgres user

(as root)
```sh
su - postgres
psql
```

```psql
\du (lists users)
\d (lists tables, roles etc)
```


