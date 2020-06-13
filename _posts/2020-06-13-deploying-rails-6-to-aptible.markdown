---
layout: post
title: "Deploying Rails 6 to aptible"
date: "2020-06-13 12:24:45 -0400"
---

I've started working on a prototype for a health care proxy innovation (to make it easier for primary care doctors to help their patients document their health care proxy wishes). I wanted to make sure that this is secure, and after a discussion with a good friend, I decided to learn how to use [Aptible](https://aptible.com) to see if I could quickly stand up a Rails application in a secure environment. I was able to do it relatively quickly (and now having done it once, I'm sure it will be much faster).

# Quickstart

[Aptibles Rails Quick start guide](https://www.aptible.com/documentation/deploy/tutorials/quickstart-guides/ruby/rails.html) was very helpful, but missed a few key steps for me. It does have links (especially to those that help you to understand the Aptible setup) so I would still recommend it.

# Install Aptible CLI

```sh
brew cask install aptible
```

Login.

```sh
aptible login
```

# Create your app and database in Aptible

You can create these with the CLI, but I used the web interface when I signed up for an account. I chose a Postgresql database.

Later, you will need the following:

* app name = the name listed in the web interfaces for apps (like the Quickstart I'll refer to this as $APP_URL)
* git repository = click on your app, and this is listed at the top as "git remote" ($GIT_REMOTE)
* database url = in the web interface, click on Databases, click on the created database ($DATABASE_URL)

# Create a test rails app

```sh
mkdir app
cd app
gem install rails
rails new . --database=postgresql
```

I add a welcome controller so I can customize the message I see.

```sh
rails generate controller Welcome index
```

Update routes.rb to include `root 'welcome#index'`

Create your databases and test it locally.

```sh
rails db:create
rails s
```

# Create your Dockerfile

I had a lot of trouble with the default Dockerfile (mostly with yarn and precompiling assets). This ended up working.

{% gist 652b85f7c4398c02fc25b18b5c32a28a Dockerfile %}

# Create your .aptible.yml

{% gist 652b85f7c4398c02fc25b18b5c32a28a .aptible.yml %}

# Set the aptible environment variables

You need to let aptible know your app name, and then any additional environment variables. 

```sh
aptible config:set --app "$APP_NAME" "DATABASE_URL=$DB_URL"
```
Note: replace $APP_NAME and $DB_URL

You also **have to set SECRET_KEY_BASE** as an environment variable otherwise the build will fail (when launching rails or precompiling assets you will get a message: secret_key_base not set for environment "production". It instructs you to set it in `rails credentials:edit`. Unfortunately it does not matter if you set it there. I suspect this is because the injected .aptible.env likely has a setting for this which is blank if you don't set it)

```sh
aptible config:set "SECRET_KEY_BASE=`rails secret`"
```

# Setup your git remote and push

```sh
git init .
git remote add aptible $GIT_REMOTE
git commit -a -m "Initial commit"
git push aptible master
```

When you push to aptible, it should then build your Docker image. This should work by now.

# Connect an endpoint

Go to the web interface for your app > Endpoints

Add a new endpoint (most of the defaults should be correct if your push worked, most importantly, it should be forwarding to your CMD 3000).

Visit the URL - you should see your welcome message.

# Save costs

To not incur the costs of a test container/db, I did the following:

* for the app - scale this to 0
* for the DB - deprovision. When you want to test this again, you will need to create a new db instance and update your DB url.



