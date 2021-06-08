---
layout: post
title: "Installing Rails 6 on EC2"
date: "2020-09-20 17:27:02 -0400"
---

So, in order to take advantage of some of the security benefits of virtual private clouds on AWS I spent the weekend getting to know a lot of AWS endpoints. There are a lot of options to explore for deploying rails on AWS including:

* Elastic Beanstalk
* aws_rails_provision
* Docker through Elastic Container Service

While there are many advantages with each of them, they also introduce new overhead and dependencies which are not always needed for smaller projects. While I hope to become more familiar with most of them so that they are easy options to choose between, I also think it is worth knowing how to go "bare metal" for your deployments which is described here: deploying rails to an EC2 instance (but within a VPC with public and private subnets and an RDS database within a private subnet).

This [GoRails](https://gorails.com/deploy/ubuntu/18.04) post was very helpful.

# Setup your EC2 instance

* Ubuntu 20.04
* t2.small (2GB)
* Place in your VPC
* Place in public subnet
* Auto-assign Public IP enable
* add to your default security group
* Make sure you have the .pem key 

Make sure you can ssh into your machine (make sure your `chmod 400 key.pem` prior to using)

```
ssh -i ~/.ssh/tutorial-docker-keypair.pem ubuntu@ec2-3-15-146-232.us-east-2.compute.amazonaws.com
```

On login, I tend to do 

```
sudo apt-get update
```

## Create a deploy user

```sh
sudo adduser deploy
```

I did use a hard password, but as noted in [this helpful post by AloucasLabs](https://www.aloucaslabs.com/miniposts/how-to-create-a-deploy-user-on-ubuntu-server-and-disable-password-authentication) we will actually disable password login.

```
sudo usermod -aG sudo deploy
```

## Add your ssh key to the server for fast login

For many deployments we could just consider using `ssh-copy-id`. Unfortunately since EC2 sets up ssh only access we need to do this manually.

This will copy your key to the clipboard (on mac).

```sh
cat ~/.ssh/id_rsa.pub | pbcopy
```

Login to the server, and for ubuntu/root update your ~/.ssh/authorized_keys file (I just used vim and paste at the end).

For the deploy user, ssh is not yet enabled, so you need to login as ubuntu, then change user, then create the appropriate directory and file (and set the proper permissions). Make sure the permissions for your .ssh files are correct as [per this article](https://community.perforce.com/s/article/6210).

As ubuntu:

```
sudo deploy
cd ~
mkdir .ssh
chmod 700 .ssh
cd .ssh
vim authorized_keys
# copy your id_rsa.pub key here
chmod 600 authorized_keys
```

Test both ssh logins as ubuntu and deploy.

Log back in as ubuntu.

## Disable password authentication

The version of sshd_config on EC2 already disables PasswordAuthentication (it is already no).

```sh
sudo vim /etc/ssh/sshd_config
```

I did uncomment

```
PubkeyAuthentication yes
```

## Disable password prompt for deploy when using sudo

```
sudo visudo
```

Add this:

```
# Deploy
deploy ALL=(ALL) NOPASSWD:ALL
```

Test your access with deploy via ssh.

**Login as deploy for the rest of the server setup.**

## Install Ruby

```
# Adding Node.js repository
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -

# Adding Yarn repository
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

# Add redis-server if you will use this
sudo add-apt-repository ppa:chris-lea/redis-server

# Refresh our packages list with the new repositories
sudo apt-get update

# Install our dependencies for compiling Ruby along with Node.js and Yarn
sudo apt-get install -y git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev software-properties-common libffi-dev dirmngr gnupg apt-transport-https ca-certificates redis-server redis-tools nodejs yarn
```

You will add the database libraries needed later.

### Install Rb-env

```
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
git clone https://github.com/rbenv/rbenv-vars.git ~/.rbenv/plugins/rbenv-vars
exec $SHELL
rbenv install 2.7.1
rbenv global 2.7.1
ruby -v
```

Install bundler

```
gem install bundler
bundle -v
```

## Install Nginx

We will use puma, but use Nginx as a reverse proxy, and for ssl (along with LetsEncrypt).

```
sudo apt-get install nginx
sudo systemctl start nginx
```

If you go to your public ip address, you should now see "Welcome to nginx"

Enable nginx to launch on reboot

```
sudo systemctl enable nginx
```

You can make sure it is running with

```sh
sudo systemctl status nginx
```

### Install Passenger

Puma is solid and simple. Passenger offers some easy with web deployment and has good capistrano plugins so we will use it here.

```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7
sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger focal main > /etc/apt/sources.list.d/passenger.list'
sudo apt-get update
sudo apt-get install -y nginx-extras libnginx-mod-http-passenger
if [ ! -f /etc/nginx/modules-enabled/50-mod-http-passenger.conf ]; then sudo ln -s /usr/share/nginx/modules-available/mod-http-passenger.load /etc/nginx/modules-enabled/50-mod-http-passenger.conf ; fi
sudo ls /etc/nginx/conf.d/mod-http-passenger.conf
```

Point passenger to the correct version of Ruby

```
sudo vim /etc/nginx/conf.d/mod-http-passenger.conf
```

Change the passenger_ruby line:

```
passenger_ruby /home/deploy/.rbenv/shims/ruby;
```

Restart nginx

```
sudo systemctl restart nginx
```

As per the [passenger deployment tutorial](https://www.phusionpassenger.com/docs/tutorials/deploy_to_production/installations/oss/aws/ruby/nginx/) you can validate the passenger installation with

```sh
sudo /usr/bin/passenger-config validate-install
```

You can check if Nginx has started the core passenger processes:

```sh
sudo /usr/sbin/passenger-memory-stats
```

Note: here I had some problems with nginx. If you start nginx with `sudo service nginx start` then systemctl does not know how to manage this. Other services such as certbot expect to manage nginx with systemctl so I recommend using it consistently. If you have ended up with nginx started by service, then you can get this fixed by either:

```sh
sudo service nginx stop
ps aux | grep nginx
```

If there are still nginx processes present:

```sh
sudo killall nginx
ps aux | grep nginx
```

### Enable your site

```
sudo rm /etc/nginx/sites-enabled/default
sudo vim /etc/nginx/sites-enabled/app
```

```
server {
  listen 80;
  listen [::]:80;

  server_name _;
  root /home/deploy/app/current/public;

  passenger_enabled on;
  passenger_app_env production;
  passenger_app_root /home/deploy/app/current;

  location /cable {
    passenger_app_group_name rails_websocket;
    passenger_force_max_concurrent_requests_per_process 0;
  }

  # Allow uploads up to 100MB in size
  client_max_body_size 100m;

  location ~ ^/(assets|packs) {
    expires max;
    gzip_static on;
  }
}
```

Reload

```
sudo systemctl reload nginx
```

# Setup your RDS instance

Follow the VPC setup for public and private subnet, with a RDS to setup a private DB subnet.

You will need the following:

* endpoint = DB_HOST
* user = postgres (unless you change it)
* password
* database name = rails_production (you get to name this in amazon setup)

```
sudo apt-get install -y libpq-dev
```


# Setup your rails app locally

```sh
 5445  mkdir metal
 
 5446  cd metal
 5447  rvm --ruby-version use 2.7.1@happi_rails --create
 5448  rvm --ruby-version use 2.7.1@happi_metal --create
  5461  gem install rails
  5462  rails new . --database=postgresql
```

Passenger enable your rails app. Remove "puma" and add the following line

```ruby
gem "passenger", ">= 5.0.25", require: "phusion_passenger/rack_handler"
```

Add capistrano to your Gemfile:

```
group :development do
  gem 'capistrano'
  gem 'capistrano-rails'
  gem 'capistrano-passenger'
  gem 'capistrano-rbenv'
end

```

```
bundle install
cap install STAGES=production
```

Edit Capfile

```ruby
require 'capistrano/rails'
require 'capistrano/passenger'
require 'capistrano/rbenv'

set :rbenv_type, :user
set :rbenv_ruby, '2.7.1'
```

Modify config/deploy.rb

```ruby
set :application, "app" 
set :repo_url, "git@github.com:narath/metal.git"

# Deploy to the user's home directory
set :deploy_to, "/home/deploy/#{fetch :application}"

append :linked_dirs, 'log', 'tmp/pids', 'tmp/cache', 'tmp/sockets', 'vendor/bundle', '.bundle', 'public/system', 'public/uploads'

# Only keep the last 5 releases to save disk space
set :keep_releases, 5

# Optionally, you can symlink your database.yml and/or secrets.yml file from the shared directory during deploy
# This is useful if you don't want to use ENV variables
append :linked_files, 'config/database.yml', 'config/master.key'

namespace :deploy do
  namespace :check do
    before :linked_files, :copy_linked_files_if_needed do
      on roles(:app), in: :sequence, wait: 10 do
        %w{master.key database.yml}.each do |config_filename|
          unless test("[ -f #{shared_path}/config/#{config_filename} ]")
            upload! "config/#{config_filename}", "#{shared_path}/config/#{config_filename}"
          end
        end
      end
    end
  end
end
```

Modify config/deploy/production.rb to point to servers IP address.

```ruby
server 'ec2-3-15-146-232.us-east-2.compute.amazonaws.com', user: 'deploy', roles: %w{app db web}
```

# Setup the environment variables

Update config/database.yml

```ruby
default: &default
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

development:
  <<: *default
  database: metal_development

test:
  <<: *default
  database: metal_test

production:
  <<: *default
  database: <%= Rails.application.credentials.dig(:production, :db_name) %>
  username: <%= Rails.application.credentials.dig(:production, :db_user) %>
  password: <%= Rails.application.credentials.dig(:production, :db_pass) %>
  host: <%= Rails.application.credentials.dig(:production, :db_host) %>
```

Update your credentials

```sh
rails credentials:edit
```

Note: if you prefer ENV variables, you can use .rbenv-vars just make sure you update these in /home/deploy/app/.rbenv-vars

```yaml
secret_key_base: YOUR_SECRET_KEY

production:
  db_user: rails
  db_pass: YOUR_RAILS_DB_PASSWORD_FROM_THE_MESSAGE_OF_THE_DAY
  db_name: your-app-name_production
  db_host: localhost
```

# Deploy

Make sure you push your repo to github.

Then deploy

```
cap production deploy
```

# Troubleshooting

## Phusion Passenger is currently not serving any applications.

I got this error message on deployment. 

I added passenger to the Gemfile.

I added `passenger_app_root /home/deploy/app/current;` to the app config.

I think the main reason was that I had not asked Nginx to serve it. So open a browser and try to view your site.

Then check that your site is one of the passenger processes with

```sh
sudo /usr/sbin/passenger-memory-stats
```

# Point your custom domain to your server

Get the IP address of your EC2 instance (this is an elastic IP)

Add your domain DNS, add an A record and point it to this

# Setup SSL using Lets Encrypt

First, edit the security group for your instance to allow HTTPS (port 443).

Then, you should make sure that the server_name is set in your site's nginx conf. If you leave it as the wildcard "_" then certbot will not be able to do the automatic installation for you.

```sh
sudo vim /etc/nginx/sites-enabled/app
```

```
  server_name app.be-happi.org;
```

Restart nginx.

```sh
sudo systemctl restart nginx
```

Then install and use certbot.

```sh
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx
```

Restart your server, and test your website to make sure it (1) redirects automatically to ssl and (2) ssl works correctly.

## Setup automatic renewals

Certbot setsup a timer that runs under systemctl to check twice daily if an update is needed, and if so to update it.

```sh
sudo systemctl status certbot.timer
```

You can also check on the systemctl timers with

```sh
systemctl list-timers
```

You should see certbot as one of the timers.

### Do a dry run

```sh
sudo certbot renew --dry-run
```

# Final checks

It might be rare, but you should check that everything works after restarting your server.

```sh
sudo shutdown -r now
```

Make sure you can deploy with db changes.

Also a good idea to make sure certbot is running otherwise you could get a surprise in 90 days.



