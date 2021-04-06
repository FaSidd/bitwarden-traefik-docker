# bitwarden-traefik-docker

Setting up [Traefik](https://github.com/traefik/traefik) reverse proxy with `docker` and running [bitwarden_rs](https://github.com/dani-garcia/bitwarden_rs) container behind it on a Raspberry Pi.

**This setup assumes you have your own domain and have `Cloudflare` DNS setup with it. 
Cloudflare will create a wildcard TLS certificate so no need for using `letsencrypt` with `traefik`.**

## Steps

### Add TLS certificate

To be able to use the `Full` mode of Cloudflare SSL you will need to have a certificate installed on
your server or what cloudflare calls `origin server` that be a self signed one or a Cloudflare one.

For my setup I used the one Cloudflare generates for me which also is a wildcard certificate for my domain.

Make sure you replace the files in the `proxy/certs` directory with your own `.pem` and `.key` files.


### Update domain names in `docker-compose.yml`

You will edit `traefik` service in the `labels:` section.

```yml
...
    labels:
      - "traefik.enable=true" # <== Enable traefik on itself to view dashboard and assign subdomain to
      - "traefik.http.routers.api.rule=Host(`monitor.domain.tld`)" # <== Setting the domain for the dashboard
      - "traefik.http.routers.api.tls=true"
      - "traefik.http.routers.api.service=api@internal" # <== Enabling the api to be a service to access
...
```

Also add your domain that you will use for bitwarden in the `labels:` section of `docker-compose.yml`.

```yaml
...
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.bitwarden-https.rule=Host(`bw.domain.tld`)"
      - "traefik.http.routers.bitwarden-https.entrypoints=web-secured"
      - "traefik.http.routers.bitwarden-https.tls=true"
      - "traefik.http.routers.bitwarden-https.service=bitwarden-https"
      - "traefik.http.services.bitwarden-https.loadbalancer.server.port=80"
      - "traefik.http.routers.bitwarden-websocket.rule=Host(`bw.domain.tld`) && Path(`/notifications/hub`)"
...
```

Once you are done adding your domains to the respective locations, save the `docker-compose.yml` file.

### Create a docker network 

Since we use an external docker network `web` we will need to create it by running the following:

```bash
docker netowrk create web
```

### Run `docker-compose.yml` file

Run the file by doing the following

```bash
docker-compose up -d
```
After this your containers should be up and running on the domains you have set.

Traefik dashboard can also be accessed locally with `http://<SERVER IP>:8080`.
