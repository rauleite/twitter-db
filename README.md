# Postgrest Docker

An `docker compose` setup for configuring a PostgreSQL database and the associated PostgREST endpoint, which is exposed to the internet using `cloudflared`.

## Initialize the containers

You can start up all the services in this project by running `docker compose up`:

```sh
# Development (localhost)
docker compose -f docker-compose.yml -f docker-compose-local.yml up

# Development (cloudflare)
docker compose -f docker-compose.yml -f docker-compose-cloud.yml up
```

## Example data

You can generate some example data by running the `create-example-data.sql` script inside of your running docker container:

```sh
# Development (localhost)
docker compose exec postgres-local psql -U user -d db -f /scripts/create-example-data.sql 

# Development (cloudflare)
docker compose exec postgres-cloud psql -U user -d db -f /scripts/create-example-data.sql
```

This will create a `users` table and a single user with the name `Raul`.

## Making requests

A `cloudflared` tunnel will be generated when you run `docker compose` up, such as `https://stanley-legal-starts-muslims.trycloudflare.com`. You can find the currently active tunnel URL by looking through the `docker compose` logs:
 
```sh
docker compose logs cloudflared | grep trycloudflare
cloudflared_1 | INFO[2021-06-14T17:26:05Z] | https://stanley-legal-starts-muslims.trycloudflare.com
```

This tunnel allows access to the PostgREST endpoint. You can make a few example requests to see data correctly returned from PostgreSQL to PostgREST, and then to your terminal:

```sh
curl https://stanley-legal-starts-muslims.trycloudflare.com/users
[{"id":1,"name":"Raul"}]

curl https://stanley-legal-starts-muslims.trycloudflare.com/users?id=eq.1
[{"id":1,"name":"Raul"}]

curl https://stanley-legal-starts-muslims.trycloudflare.com/users?id=eq.2
[]
```

You can also create records using the PostgREST API, though you should go through the [PostgREST tutorial on user authentication](https://postgrest.org/en/stable/tut1.html) to secure your API:

```sh
curl https://stanley-legal-starts-muslims.trycloudflare.com/users \
  -X POST \
  -H "Content-type: application/json" \
  -d '{"name": "Stone"}'

curl https://stanley-legal-starts-muslims.trycloudflare.com/users?name=eq.Dog
[{"id":2, "name":"Stone"}]
```

## Tunnel configuration

If you'd like to create a permanent `cloudflared` tunnel at a custom endpoint, you can [configure your tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/config). The provided `cloudflared` directory will be exposed to the Docker container, so you can add `config.yml` and any relevant JSON/certificate files to authenticate and configure your tunnel.

## Thanks

This repository and the (soon-to-come) tutorial for integrating with this on Cloudflare Workers was inspired by @eidam's great tutorial and sample codebase for running an authenticated and secure PostgreSQL, PostgREST, and `cloudflared` integration on Google Cloud, [available here](https://github.com/cloudflare/argo-tunnel-examples/tree/master/terraform-zerotrust-postgrest-worker).
