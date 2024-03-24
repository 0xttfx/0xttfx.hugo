---
title: "Novu Self Hosting"
date: 2024-03-23T18:42:59-03:00
# weight: 1
# aliases: ["/first"]
tags: ["Linux","open-source","notification infrastructure","engineering"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---

# Novu - The open-source notification infraestructure for developers.
The objective here is to show the configuration of the .env file and the NGINX reverse proxy, which was my biggest difficulty due to the lack of essential details in the project documentation for this purpose.

## How Does Novu Work?
Novu is based on  two componets

- An API for data exchange
- A panel to design notifications and their logical rules

### Architecture
Novu's Object Communication Layer - OCL is a framework that separates communication tasks into specialized components, similar to microservices, that handle specific functions, such as messaging and data management.

![architecture](/images/Novu/Novu_Architecture_v2.png)

## Let's go


- Clone Novu repo

```bash
# Go to source folder 
mkdir -p /usr/local/src/novu && cd /usr/local/src/novu

# Get the code
git clone --depth 1 https://github.com/novuhq/novu

# Go to the docker folder
cd novu/docker

# Copy the example env file
cp .env.example ./local/deployment/.env
```



- .env

```bash
# Secrets
# YOU MUST CHANGE THESE BEFORE GOING INTO PRODUCTION
# used as a secret to verify the JWT token signature
JWT_SECRET=<secret>
# used to encrypt/decrypt the provider credentials
STORE_ENCRYPTION_KEY=<key>

# Host
HOST_NAME=https://novu.yourdns.com.br

# General
# available values 'dev', 'test', 'production', 'ci', 'local'
NODE_ENV=local
MONGO_MAX_POOL_SIZE=500
MONGO_MIN_POOL_SIZE=100
# MONGO USER
MONGO_INITDB_ROOT_USERNAME=root
# MONGO PASSWORD
MONGO_INITDB_ROOT_PASSWORD=<password>
MONGO_URL=mongodb://root:<password>@mongodb:27017/novu-db?authSource=admin
REDIS_HOST=redis
REDIS_PASSWORD=<password>
REDIS_CACHE_SERVICE_HOST=redis

# AWS
S3_LOCAL_STACK=$HOST_NAME:4566
S3_BUCKET_NAME=novu-local
S3_REGION=us-east-1
AWS_ACCESS_KEY_ID=test
AWS_SECRET_ACCESS_KEY=test

# Ports
API_PORT=3000
REDIS_PORT=6379
REDIS_CACHE_SERVICE_PORT=6379
WS_PORT=3002

# Root URL
REACT_APP_WS_URL=$HOST_NAME/ws
# Uncomment this one when deploying Novu in the local environment
# as Web app local Dockerfile will have to load this to be used.
# Deployment version doesn't need as we inject it with API_ROOT_URL value.
#REACT_APP_API_URL=$HOST_NAME:3000
API_ROOT_URL=$HOST_NAME/api
DISABLE_USER_REGISTRATION=false
FRONT_BASE_URL=$HOST_NAME:4200
WIDGET_EMBED_PATH=$HOST_NAME:4701/embed.umd.min.js
WIDGET_URL=$HOST_NAME/widget

# Context Paths for reverse-proxies
GLOBAL_CONTEXT_PATH=
WEB_CONTEXT_PATH=
API_CONTEXT_PATH=api
WS_CONTEXT_PATH=ws
WIDGET_CONTEXT_PATH=widget

# Analytics
SENTRY_DSN=
# change these values
NEW_RELIC_APP_NAME=
NEW_RELIC_LICENSE_KEY=

DISABLE_USER_REGISTRATION=
``` 

- NGINX configuration

```nginx
server {
    listen 80;

    client_max_body_size 20M;

    server_name yourdns.com.br xxx.xxx.xxx.xxx;

    access_log  /var/log/nginx/novu_access.log;
    error_log /var/log/nginx/novu_error.log;

    location / {
			proxy_pass http://localhost:4200;
			proxy_http_version 1.1;
            if ($request_method = OPTIONS) {
              add_header 'Access-Control-Allow-Origin' '*';
              add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE';
              add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization';
              return 200;
            }
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $host;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
    }

    location /api {
            proxy_pass http://localhost:3000/api;
			proxy_http_version 1.1;
            add_header 'Access-Control-Allow-Origin' '*'; 
            add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE';
            add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization';
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $host;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
    }

    location /ws {
            proxy_pass http://localhost:3002/ws;
			proxy_http_version 1.1;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $host;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
    }

    location /widget {
			proxy_pass http://localhost:4500/widget;
            proxy_http_version 1.1;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $host;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
    }

    location /socket.io/ {
			proxy_pass http://localhost:3002;
            proxy_http_version 1.1;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $host;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
    }

# proxy_headers_hash_max_size 512;
# proxy_headers_hash_bucket_size 128;
}
```

Now just be happy :) 

