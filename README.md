# docker-mailserver Compose

This repository contains a fully configured docker compose environment running [docker-mailserver](https://github.com/docker-mailserver/docker-mailserver). All though docker-mailserver is a very simple and easy to setup mail server and simply get running, we have created a configuration that promotes best practices and makes it even easier to get up and running.

## Configure the server

``` sh
cp .example.env .env

docker-compose run certbot --email me@email.com -d mail.example.com
```

## Add & remove accounts

## Additional domains
