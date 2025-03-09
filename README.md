

<p align="center">
  <img src="https://raw.githubusercontent.com/BeefBytes/Assets/refs/heads/master/Other/container_illustration/v2/dockerized_paymenter.png">
</p>

# 📚 About
This guide show the quickest way of installing and running Paymenter in Docker in production. Original guide for Paymenter in Docker uses Nginx as proxy and cert generation, this guide shows how to set it up using Traefik reverse proxy which takes care of certificates generation and renewal. The original docker-compose.yml was also cleaned up by removing any exposed ports and moving all of sensitive and non sensitive variables to .env file instead. The only big difference is usage of Traefik instead of Nginx as proxy. 
Original guide by Paymenter can be found here: https://paymenter.org/docs/guides/docker/

# 🧰 Getting Started
This guide assumes you have a basic knowledge of Linux and Docker / Docker Compose.

## Requirements
- Domain
- [Docker Engine](https://docs.docker.com/engine/install/ubuntu/)
- Docker Compose (Docker Engine now comes with compose)

## Repository overview
Quick overview of directory structure, services and the ideas behind why it is setup how it is.

```
docker-paymenter
├── _base
│   ├── compose
│   │   ├── docker-compose.yml
│   │   └── .env
│   └── data
│       └── traefik
└── paymenter
    ├── compose
    │   ├── docker-compose.yml
    │   └── .env
    └── data
        ├── database
        └── paymenter
```

Both `_base` and `paymenter` directories have `compose` and `data` folders. Folder `compose` has all of configuration and variables for deployment. Folder `data` has all of the data generated by services defined inside `compose`. Such a structure clearly separates service stacks which makes it extremely modular and portable. I am not following any standards here, just something I came up with which I personally use for everything Docker related.

`_base` directory includes Traefik reverse proxy, the idea is to run services which affect the rest of other services in here. It also makes it more modular in case you're already running Traefik, which only requires you to add a docker network `paymenter` to your own Traefik container to connect rest of the services.

`paymenter` directory includes Paymenter itself, database and cache containers. Paymenter container has Traefik labels attached so it can talk to Traefik. Database and cache are both connected to Paymenter container locally and are not exposed to any other services. Paymenter traffic only goes in and out through Traefik so no ports are exposed to outside world.
# 🏗️ Installation
## Preparations
First we want configure environment variables and setup domain records before installing anything.

**Clone repository**
```bash
git clone https://github.com/EdyTheCow/docker-paymenter.git
```

**Environment variables**

Navigate to `paymenter/compose/.env/` and edit these variables.

| Variable            | Default             | Description                                                                                |
| ------------------- | ------------------- | ------------------------------------------------------------------------------------------ |
| MYSQL_PASSWORD      | -                   | Database password                                                                          |
| MYSQL_ROOT_PASSWORD | -                   | Database root password (do not reuse MYSQL_PASSWORD here, generate a long unique password) |
| DOMAIN              | pay.your-domain.com | Domain for Paymenter. This can be either with sub domain or without                        |
| APP_NAME            | Paymenter           | Your company / project name                                                                |
| APP_KEY             | -                   | Encryption key                                                                             |

The listed variables above are required to be changed to get Paymenter up and running. For generating encryption key for `APP_KEY` you can find more info at official documentation here: https://paymenter.org/docs/getting-started/installation/#install-composer

Alternatively `APP_KEY` can be randomly generated and copy / pasted from here: https://laravel-encryption-key-generator.vercel.app/

**Domain DNS records**

If you're using Cloudflare, make sure to enable the proxying by enabling the cloud icon. For full end-to-end encryption, you can also enable "Full" under the SSL/TLS section in the Cloudflare panel.

| Sub domain     | Record | Target         |
| -------------- | ------ | -------------- |
| pay.domain.com | A      | Your server IP |

## Traefik
**Set correct acme.json permissions**
Navigate to `_base/data/traefik/` and run
```bash
sudo chmod 600 acme.json
```

**Create docker network for Paymenter**
Run command to create Docker network "paymenter". This will be used by Traefik.
```bash
docker network create paymenter
```

**Start docker compose**  
Inside of `_base/compose` run
```bash
docker-compose up -d
```
## Paymenter
**Start docker compose**
Navigate to `paymenter/compose/` and run
```bash
docker compose up -d --force-recreate
docker compose run --rm paymenter php artisan migrate --force --seed
```
This will start docker containers and generate required data in the database.

**Create user**
Navigate to `paymenter/compose/` and run
```bash
docker compose run --rm paymenter php artisan p:user:create
```
# References 
- Paymenter installation guide https://paymenter.org/docs/getting-started/installation/
- Paymenter docker installation guide https://paymenter.org/docs/guides/docker/
- APP_KEY (encryption key) generation https://laravel-encryption-key-generator.vercel.app/
