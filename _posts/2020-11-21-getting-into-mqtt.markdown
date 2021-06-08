---
layout: post
title: "Getting into MQTT"
date: "2020-11-21 11:05:45 -0500"
---

Sometimes you are lucky enough to have to learn a new cool technology for a project that you are working on. I'm working on the [MakeTimeFlow](http://www.maketimeflow.com) building a juicy IOT device that helps me flow between lots of projects and boosts my executive functioning. I was using an https API endpoint to capture data, but this was heavy and slowing the devices down, and I also wanted to be able to have multiple ways that users could enter their data (device, desktop, app) so I needed some kind of pub/sub solution - and [MQTT](https://mqtt.org) was waiting. These are my notes on working with this.

# Install Mosquitto on Mac

```sh
brew install mosquitto
```

Mosquitto comes with useful command line tools for publishing and subscribing to topics

```sh
mosquitto_sub -h localhost -t test
mosquitto_pub -h localhost -t test -m "hello world"
```
# Install Mosquitto securely on the server

I based this on this great [Digital Ocean tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-the-mosquitto-mqtt-messaging-broker-on-ubuntu-18-04-quickstart). I'm using an Ubuntu 20.04 server so some things have changed.

Mosquitto is available as a snap, but it currently makes it a lot harder to work with with the documentation for learning (for example it places files not in /etc/mosquitto but in /var/snap/mosquitto/common). You can still install it with apt-get

As root

```sh
sudo apt-add-repository ppa:mosquitto-dev/mosquitto-ppa
sudo apt-get update
sudo apt-get install mosquitto mosquitto-clients
```
I like to test that it is working, so I login another terminal and in one do sub and the other pub:

```sh
mosquitto_sub -h localhost -t test
mosquitto_pub -h localhost -t test -m "hello world"
```

Make sure users have to authenticate.

```sh
sudo vim /etc/mosquitto/confi.d/default.conf
```

```conf
allow_anonymous false
password_file /etc/mosquitto/passwd
```

Create an account for you to test with.

```
sudo mosquitto_passwd -c /etc/mosquitto/passwd username
```

Now these should no longer work:

```sh
mosquitto_sub -h localhost -t test
```

But this should work:

```sh
mosquitto_sub -h localhost -t test -u "username" -P "password"
mosquitto_pub -h localhost -t test -m "hello world" -u "username" -P "password"
```
# Setup Nginx to proxy MQTT requests

I followed [this helpful guidance](https://techible.net/installing-and-testing-mosquitto-mqtt-broker/).

```sh
sudo vim /etc/mosquitto/confi.d/default.conf
```

Add in the bind_address to bind it only to localhost
```conf
allow_anonymous false
password_file /etc/mosquitto/passwd

bind_address localhost
```

I have not figured out how to do this for the websockets protocol yet, but I suspect it will be similar.

```conf
#listener 8083
#protocol websockets
```

Now the key next step is to update **nginx.conf** (note: not the normal sites-avaiable/site.conf)

```sh
sudo vim /etc/nginx/nginx.conf
```

Add this at the bottom (i.e. below the http { } )

```conf
stream {
        upstream mosquitto {
                server localhost:1883;
        }

        server {
            listen 8883 ssl;
            proxy_pass mosquitto;
            ssl_certificate /etc/letsencrypt/live/app.maketimeflow.com/fullchain.pem; # managed by Certbot
            ssl_certificate_key /etc/letsencrypt/live/app.maketimeflow.com/privkey.pem; # managed by Certbot
            #include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
            #ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
        }
}
```

Make sure mosquitto is running, restart nginx and make sure they are all running.

```sh
sudo systemctl restart mosquitto.service
systemctl status mosquitto.service
sudo systemctl restart nginx.service
systemctl status nginx.service
```

Now you should have a mosquitto reachable server through nginx.

I could not get my local `mosquitto_pub` and `_sub` commands to work, but mqtt.fx could connect with SSL/TLS > "CA signed server certificate" enabled.

# Setting up access

In the future I will use an [auth plugin](https://github.com/iegomez/mosquitto-go-auth), for now I am learning about the built in auth methods.

## File based ACL

Add in a line to your .conf

```sh
acl_file /etc/mosquitto/acl
```

Now for the acl file you can create a root, and then have all other users have access only to their flow

```conf
user youradminuser
topic readwrite #

pattern readwrite flow/%u/#
```
You can test this now, by pub and sub as admin and as users in allow and "denied" topics. You can monitor this by watching the logs:

```sh
tail -f /var/log/mosquitto/mosquitto.log
```

After adding new users or changing the acl, you can just **reload** (not restart) the service!

```sh
sudo systemctl reload mosquitto.service
```

# Appendix

## Trying to get the local commands to connect

mosquitto_pub -h example.com -t test -m "hello again" -p 8883 -u "username" -P "password" -d

To convert pem to crt
https://stackoverflow.com/questions/13732826/convert-pem-to-crt-and-key

```sh
openssl x509 -outform der -in your-cert.pem -out your-cert.crt
```

https://upcloud.com/community/tutorials/install-secure-mqtt-broker-ubuntu/

Get the trust ca from a website

```sh
openssl s_client -connect app.example.com:443
```

