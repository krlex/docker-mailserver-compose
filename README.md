# docker-mailserver Compose

This repository contains a fully configured docker compose environment running [docker-mailserver](https://github.com/docker-mailserver/docker-mailserver). All though docker-mailserver is a very simple and easy to setup mail server and simply get running, we have created a configuration that promotes best practices and makes it even easier to get up and running.

## Configure the server

``` sh
# Copy env and replace values (ONLY: HOSTNAME, POSTMASTER_ADDRESS)
cp .example.env .env

# Obtain cert for specific domain
docker run --rm -it \
  -v "${PWD}/volumes/certbot/certs/:/etc/letsencrypt/" \
  -v "${PWD}/volumes/certbot/logs/:/var/log/letsencrypt/" \
  -p 80:80 \
  certbot/certbot certonly --standalone -d mail.example.com

# Renew all certs
docker run --rm -it \
  -v "${PWD}/volumes/certbot/certs/:/etc/letsencrypt/" \
  -v "${PWD}/volumes/certbot/logs/:/var/log/letsencrypt/" \
  -p 80:80 \
  -p 443:443 \
  certbot/certbot renew

# Start docker-compose config
docker-compose up -d

# Add user
docker exec -ti mailserver setup email add postmaster@example.com
```

## Add & remove accounts

## Additional domains
