# REPRESS

A reverse proxy that passes requests to Docker containers listening on `*.repress` addresses. It also handles SSL automatically through Let's Encrypt.

## Installation

Just clone & run compose.

```bash
git clone https://github.com/BikeBearLabs/repress
cd repress
docker compose up -d --build
```

## Overview

REPRESS was created to be a super simple solution for multiple containers running on a single machine, that each had to handle a separate domain or subdomain. You don't need REPRESS if you have just a single container! Running REPRESS on its own will also do nothing except drop all incoming connections.

## Usage

In the `docker-compose.yaml` file for your container (_not_ REPRESS's!), add the following configuration to the bottom of the file (or modify your existing `networks` property):

```yaml
networks:
    private:
        driver: bridge
    public:
        name: repress
        external: true
```

This creates a network named `private` that will act as the "local" network within the compose context, & `public` which will be exposed to REPRESS.

> [!NOTE]
> You can technically name the `private` & `public` networks whatever you want, they are just named this way here to indicate their visibility to REPRESS.

To use this with your existing containers, set the unrelated services to use `private`, i.e.:

```yaml
    app:
        image: node:22.1.0-alpine
        # ...
        # add the following:
        networks:
            private:
```

& the services you want to expose as the entrypoint for the reverse proxy to use `public`, i.e.:

```yaml
    web:
        image: nginx:1.25.4-alpine
        environment:
            VIRTUAL_HOST: ${SITE_DOMAIN}
        # ...
        # add the following:
        networks:
            private:
            public:
                aliases:
                    # this will be the domain REPRESS will use to forward traffic to
                    # e.g. if it's set to `foo.com.repress`, REPRESS will forward all
                    # requests to `foo.com` to this service
                    - ${SITE_DOMAIN}.repress
```

Note the additional `aliases` property! It instructs REPRESS on what domain an incoming request must be pointing to, in order for the request to be proxied to your service. This is usually already set somewhere, like in the NGINX configuration from the example above, so just pull it in the same with the addition of the `.repress` suffix.

## Troubleshooting

### The server is dropping my connection without serving anything

This means REPRESS doesn't find a container matching the domain/subdomain you're requesting from. Double check your configuration under `networks.public.aliases`, ensuring it is an array that contains a valid `*.repress` address.

### The server is refusing my connection

Double check if REPRESS is running & that ports 80 & 443 of your server are accessible to the internet.

## License

MIT
