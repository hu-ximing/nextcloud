# Nextcloud docker compose

## Overview

This Docker compose configuration enables you to run the Nextcloud service. The service listens on a local port (in this case `8001`), using unencrypted http protocol.

This configuration is intended to run Nextcloud behind a reverse proxy with SSL encryption.

Here is an example using Nginx to forward traffic to `cloud.minglab.top` to port `8001`. Note that SSL certificates needs to be configured by the user.

Nginx server block:

```nginx
server {
    server_name  cloud.minglab.top;
    root         /var/www/cloud.minglab.top/;
    access_log   /var/log/nginx/cloud.minglab.top/access.log;
    error_log    /var/log/nginx/cloud.minglab.top/error.log;
    client_max_body_size 10G;
    proxy_request_buffering off;
    rewrite ^/\.well-known/carddav https://$server_name/remote.php/dav/ redirect;
    rewrite ^/\.well-known/caldav https://$server_name/remote.php/dav/ redirect;

    location / {
        proxy_pass http://localhost:8001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # SSL configurations goes here
}
```

## Editing configurations

- `env/db.env`
  
  pass a value to `POSTGRES_PASSWORD`

- `env/nextcloud_admin_user.txt`
  
  enter nextcloud admin username

- `env/nextcloud_admin_password.txt`
  
  enter nextcloud admin password

- `docker-compose.yml`
  
  replace `cloud.minglab.top` to desired domain name
  
  `services/nc/ports`
  
  Port listening. remove `127.0.0.1:` to listen to all hosts, otherwise it only listens locally. Change `8001` to a different number to listen on that port.

## Running containers

Run the containers

```shell
docker compose up -d
```

Stop and remove the containers

```shell
docker compose down
```

## Resolving warnings

> Your installation has no default phone region set. This is required to validate phone numbers in the profile settings without a country code. To allow numbers without a country code, please add "default_phone_region" with the respective ISO 3166-1 code of the region to your config file.

```shell
docker compose exec --user www-data app php occ config:system:set default_phone_region --value="US"
```

> The "Strict-Transport-Security" HTTP header is not set to at least "15552000" seconds. For enhanced security, it is recommended to enable HSTS as described in the security tips.

Enable `HTTP Strict Transport Security (HSTS)`

In cloudflare, go to SSL/TLS, Edge Certificates.

![HSTS](https://www.minglab.top/projects/nextcloud/HSTS.png)
