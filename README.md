# docker-mailserver Compose

This repository contains a fully configured docker compose environment running [docker-mailserver](https://github.com/docker-mailserver/docker-mailserver). All though docker-mailserver is a very simple and easy to setup mail server and simply get running, we have created a configuration that promotes best practices and makes it even easier to get up and running.

## Configure the server

This is how to do the base configuration for the mail server.

**NOTE:** be sure to set at least add the A and MX records before you start configuration.

``` sh
# Copy env and replace values (ONLY: HOSTNAME, POSTMASTER_ADDRESS)
cp .example.env .env

# Obtain cert for specific domain (CHANGE: mail.example.com)
docker run --rm -it \
  -v "${PWD}/volumes/certbot/certs/:/etc/letsencrypt/" \
  -v "${PWD}/volumes/certbot/logs/:/var/log/letsencrypt/" \
  -p 80:80 \
  certbot/certbot certonly --standalone -d mail.example.com

# Start docker-compose config
docker-compose up -d

# Add user (CHANGE: postmaster@example.com) this user is required for setup
docker exec -ti mailserver setup email add postmaster@example.com

# Generate DKIM
# After generation you can find the key in volumes/dms/config/opendkim/keys/example.com (CHANGE: example.com)
docker exec -ti mailserver setup config dkim
```

### Add DNS Records

**@** : Root domain "example.com"

**\<domain\>** : Root domain "example.com"

**\<Contents of setup config DKIM\>** : Generated DKIM key (volumes/dms/config/opendkim/keys/example.com)

| Type | Name            | Content                                             |
| ---- | --------------- | --------------------------------------------------- |
| A    | mail            | \<ipv4\>                                            |
| MX   | @               | mail.\<domain\>                                     |
| TXT  | _dmarc          | v=DMARC1; p=none; rua=mailto:postmaster@\<domain\>; |
| TXT  | mail._domainkey | \<Contents of setup config DKIM\>                   |
| TXT  | @               | v=spf1 mx ~all                                      |
| TXT  | mail            | v=spf1 mx ~all                                      |

## Connect with email client

In order to actually send emails you will need to connect an email client to the server. The following are the details to use to connect to the server. We have not included an auto configuration so this must be set manually.

**NOTE:** If using multiple domains be sure to use the root domain of the server as the host, not the domain you just added.

| Incoming (IMAP) |                 |
| --------------- | --------------- |
| protocol        | IMAP            |
| port            | 143             |
| security        | STARTTLS        |
| authentication  | Normal Password |

| Outgoing (SMTP) |                 |
| --------------- | --------------- |
| protocol        | SMTP            |
| port            | 587             |
| security        | STARTTLS        |
| authentication  | Normal Password |

## Working with the system

This is a list of commands used to work with a configured instance of docker mail server.

``` sh
# Add account
docker exec -ti mailserver setup email add \<email\> \<password\>

# Update account password
docker exec -ti mailserver setup email update \<email\> \<password\>

# Delete account
docker exec -ti mailserver setup email del \<email\> \<password\>

# List all accounts and used quotas
docker exec -ti mailserver setup email list
```

## Add other domains

You can host emails of different domains on this server. You are required to generate new SSL certs and DKIM key as well as add DNS records to the new domain to get it to work.

**NOTE:** Be sure to set at least the A and MX records for the new domain before you start adding the new domain account. When connecting with an email client be sure to use the root domain of the server as the host, not the domain you just added.

``` sh
# Obtain cert for specific domain (CHANGE: mail.example.com)
docker run --rm -it \
  -v "${PWD}/volumes/certbot/certs/:/etc/letsencrypt/" \
  -v "${PWD}/volumes/certbot/logs/:/var/log/letsencrypt/" \
  -p 80:80 \
  certbot/certbot certonly --standalone -d mail.example.com

# Add user (CHANGE: postmaster@example.com) this user is required for setup
docker exec -ti mailserver setup email add postmaster@example.com

# Generate DKIM
# After generation you can find the key in volumes/dms/config/opendkim/keys/example.com (CHANGE: example.com)
docker exec -ti mailserver setup config dkim
```

### Add DNS Records to new domain

**@** : Root domain "example.com"

**\<domain\>** : Root domain "example.com"

**\<Contents of setup config DKIM\>** : Generated DKIM key (volumes/dms/config/opendkim/keys/example.com)

| Type | Name            | Content                                             |
| ---- | --------------- | --------------------------------------------------- |
| A    | mail            | \<ipv4\>                                            |
| MX   | @               | mail.\<domain\>                                     |
| TXT  | _dmarc          | v=DMARC1; p=none; rua=mailto:postmaster@\<domain\>; |
| TXT  | mail._domainkey | \<Contents of setup config DKIM\>                   |
| TXT  | @               | v=spf1 mx ~all                                      |
| TXT  | mail            | v=spf1 mx ~all                                      |

## Renew all SSL certificates

``` bash
# Renew all certs
docker run --rm -it \
  -v "${PWD}/volumes/certbot/certs/:/etc/letsencrypt/" \
  -v "${PWD}/volumes/certbot/logs/:/var/log/letsencrypt/" \
  -p 80:80 \
  -p 443:443 \
  certbot/certbot renew
```

## Enable Roundcube webmail containers

We have included the necessary configuration to run a Roundcube webmail service in addition ot the mailserver itself. By default we do not have Roundcube enabled, this is because docker-mailserver is the main focus of this configuration and we assume that most users want to keep the mail server as lightweight and simple as possible. The mailserver runs as a single container with no DB but to run Roundcube it requires its own database and an nginx configuration. 

To enable Roundcube simply uncomment the 3 required containers in the compose file, Add "DB_PASSWORD" in the .env and finally copy the nginx config and change the example domain to the actual domain you are using (mail.example.com).

``` bash
docker-compose down
cp ./config/nginx/example.nginx.conf ./config/nginx/nginx.conf
docker-compose up -d
```

## Web admin interface

We currently do not have a web admin interface to manage the creation of users but this is something we are developing. We hope to contribute this feature directly to [docker-mailserver](https://github.com/docker-mailserver/docker-mailserver) in the future.
