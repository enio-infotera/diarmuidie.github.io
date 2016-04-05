---
layout: post
title: 'Using Nginx as a Reverse Proxy for Developing Microservices with Docker Compose'
tags:
    - Article
excerpt: "I've recently started using Docker for my development environment. One of the first problems I ran into was how to run multiple Docker Compose microservice projects on the same host if they all need to run on the same port (port 80 for example).

I will outline one of the solutions that involves using Nginx as a reverse proxy to send requests to the correct backend microservice."
---

## Introduction

I've recently started using Docker for my development environment. One of the first problems I ran into was how to run multiple Docker Compose microservice projects on the same host if they all need to run on the same port (port 80 for example).

I will outline one of the solutions that involves using Nginx as a reverse proxy to send requests to the correct backend microservice.

![Docker Compose Reverse Proxy Layout](/media/docker-reverse-proxy/nginx-docker-reverse-proxy.png)

The code for this example is available on [Github](https://github.com/diarmuidie/docker-compose-reverse-proxy-example/)

## The Setup
Each microservice project will be a standalone docker compose project with its own `docker-composer.yml` file. For this example we will have two microservices:

 - microservice1 [Link](https://github.com/diarmuidie/docker-compose-reverse-proxy-example/tree/master/microservice1)
 - microservice2 [Link](https://github.com/diarmuidie/docker-compose-reverse-proxy-example/tree/master/microservice2)

(Notice that the microservices don't bind to any external ports, the reverse proxy will handle this for us).

For simplicity these microservices return static HTML using a nginx container (in real life they could be fully featured PHP/Python/Go etc. apps).

We want to be able to connect to the microservice on our local machine using the `microservice1.test` and `microservice2.test` domains.

## Configuring The Proxy
Now that we have our microservices setup we can look at configuring the nginx reverse proxy.

### Docker Compose
First thing we need to do is create a new docker-compose project for the proxy with the following `docker-compose.yml` file:

```yml
version: '2'
services:
  proxy:
    build: ./
    networks:
      - microservice1
      - microservice2
    ports:
      - 80:80
      - 443:443

networks:
  microservice1:
    external:
      name: microservice1_default
  microservice2:
    external:
      name: microservice2_default
```
This file tells docker-compose to create a `proxy` service that connects to the external microservice project networks. It needs to be able to connect to thee networks so that it can proxy the requests it receives to them. We also bind the proxy service to the hosts port 80 and 443 so that we can connect to it.

### Nginx

We now need to  configure Nginx to forward traffic to the correct backend microservice. Each microservice needs its own server block like so:

```nginx
server {
    listen 80;
    server_name microservice1.test;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
        proxy_request_buffering off;
        proxy_http_version 1.1;
        proxy_intercept_errors on;
        proxy_pass http://microservice1_app_1;
    }

    access_log off;
    error_log  /var/log/nginx/error.log error;
}
```
This block tells nginx to pass all requests for `microservice1.test` to the microservices app container (`http://microservice1_app_1`). You will get this domain when you start the microservice container by running `docker ps` and looking under the "NAMES" column.

Note that the `proxy_intercept_errors` option is set to `on` so that nginx will return all error responses from the microservice, instead of returning the default Nginx response. This is useful if you return debug info as part of the response.

In the example default config on [Github](https://github.com/diarmuidie/docker-compose-reverse-proxy-example/blob/master/proxy/default.conf) I've added some more options to enable SSL and abstract out the proxy config.

## Using the Reverse Proxy

Now that we have our microservices and proxy setup we can run them all to start developing.

1. Start each of the microservices first:
  `cd microservice1`
  `docker-compose up -d`
  Repeat for each microservice needed in your project.

2. Now you can start the proxy:
  `cd ../proxy`
  `docker-compose up -d`

3. Add the domains to your `/etc/hosts` file:
  `echo "192.168.99.100 microservice1.test" >> /etc/hosts`
  `echo "192.168.99.100 microservice2.test" >> /etc/hosts`

You're now able to access both the microservices on port 80 using their domains (http://microservice1.test/ etc.).

## Conclusion
This method involves some initial setup to get the proxy working. The main pain-point is that you need to remember to start the proxy and each microservice each time you want to use them, but this could be fixed by automating it.

This is only a brief overview, have a look a the working example on Github for more detail:  [https://github.com/diarmuidie/docker-compose-reverse-proxy-example/](https://github.com/diarmuidie/docker-compose-reverse-proxy-example/)
