# Docker-CGWire

Docker compose for [Kitsu](https://kitsu.cg-wire.com/) and [Zou](https://zou.cg-wire.com/)

## Notice

This project is a detached version of [MathieuBOUZARD's](https://gitlab.com/mathbou) cgwire automation project on [gitlab](https://gitlab.com/mathbou/docker-cgwire). Please make sure to check out that project as well.

## About CGWire

CGWire is an open-source project management tool designed for animation and VFX studios. It consists of two main components:

- **Kitsu**: A web-based project management tool for tracking animation production
- **Zou**: The API server and database backend for Kitsu

## Changes for eefafx

We are using the same flow and same docker containers as the original project, with some minor changes to make the deployment process easier:

- Reading environment variables from `.env` file
- Sourcing the `.env` file at the start of scripts
- Added a `.env.sample` file to follow along easily
- Improved documentation and deployment scripts

## Prerequisites

Before running this project, make sure you have the following installed:

- Docker and Docker Compose
- `curl` (for fetching version information)
- `jq` (for JSON parsing)
- `git` (for cloning repositories)

## Quick Start

### 1. Setup Environment

First, copy the sample environment file and customize it:

```bash
cp .env.sample .env
```

Edit `.env` file with your specific configuration (database passwords, mail settings, etc.).

### 2. Run the Stack

Make the build script executable and run it:

```bash
chmod +x build.sh
./build.sh local
```

This will start the following docker containers, and you should see something like this:
```
CONTAINER ID   IMAGE                         COMMAND                  CREATED      STATUS        PORTS                    NAMES
<id>           kitsu:0.20.69                 "/docker-entrypoint.…"   1 days ago   Up 1 days   0.0.0.0:80->80/tcp         eefa-kitsu
<id>           zou:0.20.57                   "gunicorn --error-lo…"   1 days ago   Up 1 days                              eefa-zou-event
<id>           zou:0.20.57                   "gunicorn --error-lo…"   1 days ago   Up 1 days   5000->5000/tcp             eefa-zou-app
<id>           postgres:15-alpine            "docker-entrypoint.s…"   1 days ago   Up 1 days   5432->5432/tcp             eefa-db-15
<id>           redis:alpine                  "docker-entrypoint.s…"   1 days ago   Up 1 days   6379/tcp                   eefa-redis
<id>           zou:0.20.57                   "rq worker -c zou.jo…"   1 days ago   Up 1 days                              eefa-zou-jobs
<id>           getmeili/meilisearch:v1.1.1   "tini -- /bin/sh -c …"   1 days ago   Up 1 days   7700/tcp                   eefa-indexer
```

### 3. Create Admin User from ZOU
In order to create an admin user use the following commands to create an admin user in the zou backend:
```bash
docker compose exec -it zou-app bash
zou create-admin <email_of_choice> -p <password>
```

#### Tip
You can use `zou --help` in the container to find some useful commands to manage and make some changes in the zou-app back-end if needed.

### 4. Setup indexer
First of all make sure that the indexer (meilisearch) container is running correctly, it's tricky and can have some issue with the indexer key,
once made sure that the indexer is running just fine init the index data using the following command in zou-app container once again:
```bash
docker compose exec -it zou-app bash
zou reset-search-index
```

This will run the collections creation command in the meilisearch container and you can search in the kitsu app perfectly.


### 5. Extra
if you encounter server error there might be an issue with the db, you can resolve them using the following commands in the zou-app container:
```bash
docker compose exec -it zou-app bash
zou init-db
```
optionally you can run the following command to initiate some data for files and shots and etc.
```bash
zou init-data
```


For CGWire-specific issues, please refer to:
- [Kitsu Documentation](https://kitsu.cg-wire.com/)
- [Zou Documentation](https://zou.cg-wire.com/)
- [CGWire Community](https://discord.gg/VbCxtKN)
