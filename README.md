<!-- DO NOT EDIT THIS FILE MANUALLY  -->

# [imagegenius/immich](https://github.com/imagegenius/docker-immich)

[![GitHub Release](https://img.shields.io/github/release/imagegenius/docker-immich.svg?color=007EC6&labelColor=555555&logoColor=ffffff&style=for-the-badge&logo=github)](https://github.com/imagegenius/docker-immich/releases)
[![GitHub Package Repository](https://shields.io/badge/GitHub%20Package-blue?logo=github&logoColor=ffffff&style=for-the-badge)](https://github.com/imagegenius/docker-immich/packages)
[![Jenkins Build](https://img.shields.io/jenkins/build?labelColor=555555&logoColor=ffffff&style=for-the-badge&jobUrl=https%3A%2F%2Fci.imagegenius.io%2Fjob%2FDocker-Pipeline-Builders%2Fjob%2Fdocker-immich%2Fjob%2Fmain%2F&logo=jenkins)](https://ci.imagegenius.io/job/Docker-Pipeline-Builders/job/docker-immich/job/main/)
[![IG CI](https://img.shields.io/badge/dynamic/yaml?color=007EC6&labelColor=555555&logoColor=ffffff&style=for-the-badge&label=CI&query=CI&url=https%3A%2F%2Fci-tests.imagegenius.io%2Fimmich%2Flatest-main%2Fci-status.yml)](https://ci-tests.imagegenius.io/immich/latest-main/index.html)

[Immich](https://immich.app/) is a high performance self-hosted photo and video backup solution.

[![immich](https://user-images.githubusercontent.com/27055614/182044984-2ee6d1ed-c4a7-4331-8a4b-64fcde77fe1f.png)](https://immich.app/)

## Supported Architectures

We use Docker manifest for cross-platform compatibility. More details can be found on [Docker's website](https://github.com/docker/distribution/blob/master/docs/spec/manifest-v2-2.md#manifest-list).

To obtain the appropriate image for your architecture, simply pull `ghcr.io/imagegenius/immich:latest`. Alternatively, you can also obtain specific architecture images by using tags.

This image supports the following architectures:

| Architecture | Available | Tag |
| :----: | :----: | ---- |
| x86-64 | ✅ | amd64-\<version tag\> |
| arm64 | ✅ | arm64v8-\<version tag\> |
| armhf | ❌ | |

## Version Tags

This image offers different versions via tags. Be cautious when using unstable or development tags, and read their descriptions carefully.

| Tag | Available | Description |
| :----: | :----: |--- |
| latest | ✅ | Latest Immich release with an Ubuntu base. |
| noml | ✅ | Latest Immich release with an Alpine base. Machine-learning is completely removed, which makes for a very lightweight image. |
## Application Setup

The WebUI can be accessed at `http://your-ip:8080` Follow the wizard to set up Immich.

To use Immich, you need to have PostgreSQL 14/15 server set up externally, and Redis set up externally or within the container using a docker mod.

To set up redis using the docker mod, use the following:

Set `DOCKER_MODS=imagegenius/mods:universal-redis`, and `REDIS_HOSTNAME` to `localhost`.

When `CUDA_ACCELERATION` is set to `true`, container startup times will be increased, as it will force upgrade the cuda pip packages every restart. TAG OBJECTS and ENCODE CLIP jobs will use the GPU acceleration, however RECOGNIZE FACES job won't use the GPU acceleration. If you want to have GPU acceleration for all your machine-learning jobs, you have to setup a new immich-cuda-node docker. All instructions to set it up can be found on [the repo](https://github.com/imagegenius/docker-immich-cuda-node/)

## Usage

Example snippets to start creating a container:

### Docker Compose

```yaml
---
version: "2.1"
services:
  immich:
    image: ghcr.io/imagegenius/immich:latest
    container_name: immich
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - DB_HOSTNAME=192.168.1.x
      - DB_USERNAME=postgres
      - DB_PASSWORD=postgres
      - DB_DATABASE_NAME=immich
      - REDIS_HOSTNAME=192.168.1.x
      - DISABLE_MACHINE_LEANRNING=false #optional
      - DISABLE_TYPESENSE=false #optional
      - DB_PORT=5432 #optional
      - REDIS_PORT=6379 #optional
      - REDIS_PASSWORD= #optional
      - CUDA_ACCELERATION=false #optional
    volumes:
      - path_to_appdata:/config
      - path_to_photos:/photos
      - path_to_machine-learning:/config/machine-learning #optional
    ports:
      - 8080:8080
    restart: unless-stopped
# This container requires an external application to be run separately to be run separately.
# Redis:
  redis:
    image: redis
    ports:
      - 6379:6379
    container_name: redis
# PostgreSQL 14:
  postgres14:
    image: postgres:14
    ports:
      - 5432:5432
    container_name: postgres14
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: immich
    volumes:
      - path_to_postgres:/var/lib/postgresql/data


```

### Docker CLI ([Click here for more info](https://docs.docker.com/engine/reference/commandline/cli/))

```bash
docker run -d \
  --name=immich \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Etc/UTC \
  -e DB_HOSTNAME=192.168.1.x \
  -e DB_USERNAME=postgres \
  -e DB_PASSWORD=postgres \
  -e DB_DATABASE_NAME=immich \
  -e REDIS_HOSTNAME=192.168.1.x \
  -e DISABLE_MACHINE_LEANRNING=false `#optional` \
  -e DISABLE_TYPESENSE=false `#optional` \
  -e DB_PORT=5432 `#optional` \
  -e REDIS_PORT=6379 `#optional` \
  -e REDIS_PASSWORD= `#optional` \
  -e CUDA_ACCELERATION=false `#optional` \
  -p 8080:8080 \
  -v path_to_appdata:/config \
  -v path_to_photos:/photos \
  -v path_to_machine-learning:/config/machine-learning `#optional` \
  --restart unless-stopped \
  ghcr.io/imagegenius/immich:latest

# This container requires an external application to be run separately.
# Redis:
docker run -d \
  --name=redis \
  -p 6379:6379 \
  redis

# PostgreSQL 14:
docker run -d \
  --name=postgres14 \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=immich \
  -v path_to_postgres:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:14


```

## Variables

To configure the container, pass variables at runtime using the format `<external>:<internal>`. For instance, `-p 8080:80` exposes port `80` inside the container, making it accessible outside the container via the host's IP on port `8080`.

| Variable | Description |
| :----: | --- |
| `-p 8080` | WebUI Port |
| `-e PUID=1000` | UID for permissions - see below for explanation |
| `-e PGID=1000` | GID for permissions - see below for explanation |
| `-e TZ=Etc/UTC` | Specify a timezone to use, see this [list](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List). |
| `-e DB_HOSTNAME=192.168.1.x` | PostgreSQL Host |
| `-e DB_USERNAME=postgres` | PostgreSQL Username |
| `-e DB_PASSWORD=postgres` | PostgreSQL Password |
| `-e DB_DATABASE_NAME=immich` | PostgreSQL Database Name |
| `-e REDIS_HOSTNAME=192.168.1.x` | Redis Hostname |
| `-e DISABLE_MACHINE_LEANRNING=false` | Set to `true` to disable machine learning |
| `-e DISABLE_TYPESENSE=false` | Set to `true` to disable Typesense (disables searching completely!) |
| `-e DB_PORT=5432` | PostgreSQL Port |
| `-e REDIS_PORT=6379` | Redis Port |
| `-e REDIS_PASSWORD=` | Redis password |
| `-e CUDA_ACCELERATION=false` | Set to `true` to enable CUDA Acceleration (NVIDIA GPU must be passed through! `--gpus=all`) |
| `-v /config` | Contains the logs, machine-learning models and Typesense data |
| `-v /photos` | Contains all the photos uploaded to Immich |
| `-v /config/machine-learning` | Store the machine-learning models (~800MB) |

## Umask for running applications

All of our images allow overriding the default umask setting for services started within the containers using the optional -e UMASK=022 option. Note that umask works differently than chmod and subtracts permissions based on its value, not adding. For more information, please refer to the Wikipedia article on umask [here](https://en.wikipedia.org/wiki/Umask).

## User / Group Identifiers

To avoid permissions issues when using volumes (`-v` flags) between the host OS and the container, you can specify the user (`PUID`) and group (`PGID`). Make sure that the volume directories on the host are owned by the same user you specify, and the issues will disappear.

Example: `PUID=1000` and `PGID=1000`. To find your PUID and PGID, run `id user`.

```bash
  $ id username
    uid=1000(dockeruser) gid=1000(dockergroup) groups=1000(dockergroup)
```

## Updating the Container

Most of our images are static, versioned, and require an image update and container recreation to update the app. We do not recommend or support updating apps inside the container. Check the [Application Setup](#application-setup) section for recommendations for the specific image.

Instructions for updating containers:

### Via Docker Compose

* Update all images: `docker-compose pull`
  * or update a single image: `docker-compose pull immich`
* Let compose update all containers as necessary: `docker-compose up -d`
  * or update a single container: `docker-compose up -d immich`
* You can also remove the old dangling images: `docker image prune`

### Via Docker Run

* Update the image: `docker pull ghcr.io/imagegenius/immich:latest`
* Stop the running container: `docker stop immich`
* Delete the container: `docker rm immich`
* Recreate a new container with the same docker run parameters as instructed above (if mapped correctly to a host folder, your `/config` folder and settings will be preserved)
* You can also remove the old dangling images: `docker image prune`

## Versions

* **23.05.23:** - move to ubuntu lunar and support cuda acceleration for machine-learning
* **22.05.23:** - deprecate postgresql docker mod
* **18.05.23:** - add support for facial recognition
* **07.05.23:** - remove unused `JWT_SECRET` env
* **13.04.23:** - add variables to disable typesense and machine learning
* **10.04.23:** - fix gunicorn
* **04.04.23:** - use environment variables to set location of the photos folder
* **09.04.23:** - Cache is downloaded to the host (/config/transformers)
* **01.04.23:** - remove unused Immich environment variables
* **21.03.23:** - Add service checks
* **05.03.23:** - add typesense
* **27.02.23:** - re-enable aarch64 with pre-release torch build
* **18.02.23:** - use machine-learning with python
* **11.02.23:** - use external app block
* **09.02.23:** - Use Immich environment variables for immich services instead of hosts file
* **09.02.23:** - execute CLI with the command immich
* **04.02.23:** - shrink image
* **26.01.23:** - add unraid migration to readme
* **26.01.23:** - use find to apply chown to /app, excluding node_modules
* **26.01.23:** - enable ci testing
* **24.01.23:** - fix services starting prematurely, causing permission errors.
* **23.01.23:** - add noml image to readme and add aarch64 image to readme, make github release stable
* **21.01.23:** - BREAKING: Redis is removed. Update missing param_env_vars & opt_param_env_vars for redis & postgres
* **02.01.23:** - Initial Release.
