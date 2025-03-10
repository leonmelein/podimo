# Podimo to RSS

Podimo is a proprietary podcasting player that enables you to listen to various exclusive shows behind a paywall.
This tool allows you to stream Podimo podcasts with your preferred podcast player, without having to use the Podimo app.

## Docker environment variables

* **BIND_HOST**: Sets the host and port to bind on (default: 127.0.0.1:12104)
* **HOST**: Host that will be displayed in the example. (default: podimo.thijs.sh)

## Usage
To obtain a Podimo RSS feed, you need to provide
* Your Podimo username
* Your Podimo password
* The ID of the podcast you want to listen to

These values are passed via an URL.
### Example
------------
* **Username** `example@example.com`
* **Password** `this-is-my-password`
* **Podcast ID** `12345-abcdef`

The URL will be
`https://podimo.thijs.sh/feed/example%40example.com/this-is-my-password/12345-abcdef.xml`. You can pass this URL directly to your favorite podcast player that supports RSS feeds.

## Installation for self-hosting
If you want run this script yourself, you need a recent Python 3 version and install the packages in `requirements.txt` with `pip install -r requirements.txt`.

## Privacy
The script keeps track of a few things in memory:
- Your username and password, used to login and to create an access token. This is only used temporarily during a request itself.
- A cryptographic hash that is calculated based on your username and password.
- A Podimo access token, which is kept in memory for accessing pages after logging in.

This data is _never_ written to the disk and it is _never_ logged. The hosted script runs behind an `nginx` reverse proxy that requires HTTPS and does not keep any `access_logs`. The `nginx` configuration is:

```nginx
server {
	server_name podimo.thijs.sh;
	access_log off;

	location / {
        proxy_pass http://127.0.0.1:12104;

	    add_header Strict-Transport-Security "max-age=31536000;" always;

	    add_header X-Accel-Buffering no;
	    proxy_buffering off;

        proxy_set_header X-Real-IP $remote_addr;
	}

    listen [::]:443 ssl http2 ipv6only=on; # managed by Certbot
    listen 443 ssl http2; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/podimo.thijs.sh/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/podimo.thijs.sh/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}


server {
    if ($host = podimo.thijs.sh) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

	access_log off;
	listen 80;
	listen [::]:80;
	server_name podimo.thijs.sh;
    return 404; # managed by Certbot
}
```

# License
```
Copyright 2022 Thijs Raymakers

Licensed under the EUPL, Version 1.2 or – as soon they
will be approved by the European Commission - subsequent
versions of the EUPL (the "Licence");
You may not use this work except in compliance with the
Licence.
You may obtain a copy of the Licence at:

https://joinup.ec.europa.eu/software/page/eupl

Unless required by applicable law or agreed to in
writing, software distributed under the Licence is
distributed on an "AS IS" basis,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either
express or implied.
See the Licence for the specific language governing
permissions and limitations under the Licence.
```
