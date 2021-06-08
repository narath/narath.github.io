---
layout: post
title: "Deploying Rails 6 to AWS with Docker"
date: "2020-09-20 16:48:19 -0400"
---

[This post](https://dev.to/farleyknight/how-to-deploy-a-rails-application-to-aws-with-docker-2fnc) by Farley Knight was really helpful in getting to know how to Dockerize and then deploy your rails app to AWS. Here are the settings that worked for me.

# Dockerize and prepare your container

Dockerfile

```
FROM ruby:2.7.1-slim

# Update packages
RUN apt-get update -qq

# Install needed components
RUN apt-get install -y \
    build-essential ca-certificates vim curl \
    libpq-dev

# Get the latest node and yarn
RUN curl -sL https://deb.nodesource.com/setup_10.x | bash - && \
  curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
  echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
  apt-get update && apt-get install -y nodejs yarn

# once working, consider cleaning up files
#  && rm -rf /var/lib/apt/lists/*

ENV APP_HOME /app
ENV RAILS_SERVE_STATIC_FILES true
ENV RAILS_LOG_TO_STDOUT true

ADD . ${APP_HOME}
WORKDIR ${APP_HOME}

COPY Gemfile .
COPY Gemfile.lock .
RUN gem update bundler
# RUN bundle install --jobs 5 --without development test
RUN bundle install --jobs 5

# Copy my project over
COPY . ${APP_HOME}

# COPY package.json .
# COPY yarn.lock .
RUN yarn install --check-files

EXPOSE 8080

ENTRYPOINT ["sh", "./entrypoint.sh"]

```sh
#!/usr/bin/env bash

# Precompile assets
bundle exec rails assets:precompile
echo "Assets compiled."

# bundle exec yarn install --check-files

# If the database exists, migrate. Otherwise setup
bundle exec rails db:migrate 2>/dev/null | bundle exec rails db:create db:migrate
echo "Database migrated."

# Remove any old pre-existing pids for Rails
rm -f tmp/pids/server.pid

bundle exec rails server -b 0.0.0.0 -p 8080
```

```yaml

version: '3.1'

services: 
  db:
    image: postgres:12.4
    restart: always
    environment:
      POSTGRES_PASSWORD: SOMETHING_LONG_AND_SECURE

  web:
    depends_on:
      - db
    build: .
    ports:
      - "8080:8080"
    environment:
      DB_HOST: db
      DB_PORT: 5432
      DB_USER: postgres
      DB_PASSWORD: SOMETHING_LONG_AND_SECURE
      RAILS_ENV: development
      RAILS_MAX_THREADS: 5
    volumes:
      - ".:/app"

volumes:
  db:
```

.dockerignore

```
.dockerignore
.git
log/
tmp/
```

## Build and test locally

```sh
docker-compose build
docker-compose up
```

# Setup AWS

## Setup an elastic container registry.

Tag your web docker container.

Login to ecr.

Push your image.

## Create your ECS cluster

Create your cluster.

Create a task. You define the container in here with the deployment environment config, so make sure you set your environment variables.

```json

{
  "ipcMode": null,
  "executionRoleArn": null,
  "containerDefinitions": [
    {
      "dnsSearchDomains": null,
      "environmentFiles": null,
      "logConfiguration": {
        "logDriver": "awslogs",
        "secretOptions": null,
        "options": {
          "awslogs-group": "/ecs/$TASKNAME",
          "awslogs-region": "us-east-2",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "entryPoint": null,
      "portMappings": [
        {
          "hostPort": 80,
          "protocol": "tcp",
          "containerPort": 8080
        }
      ],
      "command": [
        "sh",
        "entrypoint.sh"
      ],
      "linuxParameters": null,
      "cpu": 0,
      "environment": [
        {
          "name": "DB_HOST",
          "value": "$RDS_DB_ENTRYPOINT"
        },
        {
          "name": "DB_PASSWORD",
          "value": "$LONGPASSWORD"
        },
        {
          "name": "DB_PORT",
          "value": "5432"
        },
        {
          "name": "DB_USER",
          "value": "postgres"
        },
        {
          "name": "RAILS_ENV",
          "value": "production"
        },
        {
          "name": "SECRET_KEY_BASE",
          "value": "$USE_RAILS_SECRET_TO_GENERATE_A_NEW_SECRET"
        }
      ],
      "resourceRequirements": null,
      "ulimits": null,
      "dnsServers": null,
      "mountPoints": [],
      "workingDirectory": null,
      "secrets": null,
      "dockerSecurityOptions": null,
      "memory": null,
      "memoryReservation": null,
      "volumesFrom": [],
      "stopTimeout": null,
      "image": "$COPY_ARN_FROM_YOUR_DOCKER_IMAGE_IN_ECR",
      "startTimeout": null,
      "firelensConfiguration": null,
      "dependsOn": null,
      "disableNetworking": null,
      "interactive": null,
      "healthCheck": null,
      "essential": true,
      "links": null,
      "hostname": null,
      "extraHosts": null,
      "pseudoTerminal": null,
      "user": null,
      "readonlyRootFilesystem": null,
      "dockerLabels": null,
      "systemControls": null,
      "privileged": null,
      "name": "web5"
    }
  ],
  "placementConstraints": [],
  "memory": "512",
  "taskRoleArn": null,
  "compatibilities": [
    "EC2"
  ],
  "taskDefinitionArn": "arn:aws:ecs:us-east-2:569876213943:task-definition/$TASKNAME:1",
  "family": "$TASKNAME",
  "requiresAttributes": [
    {
      "targetId": null,
      "targetType": null,
      "value": null,
      "name": "com.amazonaws.ecs.capability.logging-driver.awslogs"
    },
    {
      "targetId": null,
      "targetType": null,
      "value": null,
      "name": "com.amazonaws.ecs.capability.ecr-auth"
    },
    {
      "targetId": null,
      "targetType": null,
      "value": null,
      "name": "com.amazonaws.ecs.capability.docker-remote-api.1.19"
    }
  ],
  "pidMode": null,
  "requiresCompatibilities": [
    "EC2"
  ],
  "networkMode": null,
  "cpu": "1024",
  "revision": 1,
  "status": "ACTIVE",
  "inferenceAccelerators": null,
  "proxyConfiguration": null,
  "volumes": []
  }
```

Then create a service in the cluster.

If it works correctly, after creating the service, it will start and create an image.

## To login to your server

Make sure your routes support SSH, and your machines are connected to your public subnet.

Make sure you got the .pem file (and make sure chmod 0400 the file otherwise it will not work for ssh).

Get the endpoint of your ECS instance

```
ssh -i downloaded.pem ec2-user@ecID.us-east-2.compute.amazonaws.com
```

You use the ec2-user to login to the docker host (Amazon Linux 2, remember to use yum for updates).

To access your docker file

```sh
docker ps
docker exec -it IMAGE_ID /bin/bash
```

# How to push an update to your server

After updating and testing your code locally, you update the AWS server, but forcing a new deployment of the service.

Build your new docker image

```sh
docker-compose build
docker-compose up
```
Docker Compose tags it with latest, you need to tag your destination aws tag with latest as well

```sh
docker tag docker1_web:latest 569876213943.dkr.ecr.us-east-2.amazonaws.com/tutorial:latest
```

Push it to aws

```sh
docker push 569876213943.dkr.ecr.us-east-2.amazonaws.com/tutorial:latest
```

Then ask for an update service.

As noted in [this stack overflow post](https://stackoverflow.com/questions/34840137/how-do-i-deploy-updated-docker-images-to-amazon-ecs-tasks) this will only work if you change the service to allow the Minimum Healthy Percent to be 0, which will let it shut down your instance and deploy the next one (as noted: this can create some downtime). If you have a load balancer then you can let the number of tasks be greater than 1 which will allow it to swap as well.

aws ecs update-service --cluster <cluster name> --service <service name> --force-new-deployment

aws ecs update-service --cluster tutorial-cluster5 --service tutorial-docker-service5 --force-new-deployment
