# laravel-docker-platform
Docker container for Laravel and PHP projects.  Graceful shutdown.  No-python supervisor. No-SSH.

# Customizing
You can add your own one-time, start up scripts to `/platform/start-container.d/`

If any script in `start-container.d` exists with a return code other than 0, the
container will fail to start.

You can change any PHP ini by bind mounting a file to `/usr/local/etc/php-fpm.d/` or
using your container orchestrator's config file functionality.

# Available run-time environment variables

  * LARAVEL_WORK_DIR  (defaults to /app)
  * LARAVEL_HORIZON_WORK_DIR (defaults to /app)
  * LARAVEL_QUEUE_WORK_DIR (defaults to /app)
  * LARAVEL_MIGRATE_COMMAND  (defaults to "/usr/local/bin/php artisan migrate --force")
  * ENABLE_MIGRATIONS
  * ENABLE_CONFIG_CACHE
  * ENABLE_CHANGE_OWNER
  * ENABLE_ARTISAN_SCHEDULE

# Config cache
One of the start-up scripts will cache the config with artisan.  You can enable
this step by setting the environment variable
`ENABLE_CONFIG_CACHE` to any value.

# Running queue:work as a standalone container
The script `/platform/start-queue` can be used to start artisan `queue:work`

set `LARAVEL_QUEUE_WORK_COMMAND` with your desired_commands

```
---
version: '3.7'

services:
  laravel:
    image:  your-docker-repo/your-app-name:latest-prod
  queue:
    image:  markkimsal/php-platform:8.0-nginx-fpm
    command: /platform/start-queue

    environment:
      LARAVEL_QUEUE_WORK_DIR: /app
      LARAVEL_QUEUE_WORK_COMMAND: /usr/local/bin/php artisan queue:work default --tries=1 --sleep=3
```


# Running horizon as a standalone container

The script `/platform/start-horizon` can be used to call only artisan:horizon and no other processes.
You can deploy the same built app container with 2 different `command` values to get 2 different
container behaviors.

```
---
version: '3.7'
networks:
  traefik_ingress:
    external: true

services:
  laravel:
    image:  your-docker-repo/your-app-name:latest-prod
    networks:
      - 'traefik_ingress'

    deploy:
      endpoint_mode: dnsrr
      mode: replicated
      replicas: 3
      labels:
        - "traefik.enable=true"
        - "traefik.docker.network=traefik_ingress"
        - "traefik.http.routers.laraveltest.rule=Host(`app.localhost.test`)"
        - "traefik.http.routers.laraveltest.entrypoints=websecure"
        - "traefik.http.routers.laraveltest.tls=true"
        - "traefik.http.routers.laraveltest.tls.certresolver=myresolver"
        - "traefik.http.routers.laraveltest.service=laraveltest"
        - "traefik.http.services.laraveltest.loadbalancer.server.port=8080"

  horizon:
    image:  your-docker-repo/your-app-name:latest-prod
    command: /platform/start-horizon

    deploy:
      endpoint_mode: dnsrr
      mode: replicated
      replicas: 2
    environment:
      LARAVEL_HORIZON_WORK_DIR: /app
```

# Running horizon as supervised service
If you want to run horizon under the supervisor `runit` you can add an environment variable `START_HORIZON_AS_SERVICE=yes` to your docker container by any means.


# WWWUID and WWWGID
You can change the id of the `www-data` user to match any owner of files on your production system.

On your local development system, you can set WWWUID and WWWGID to 1000 so there are no ownership
problems between your code and the running www-data user.  (Note, this is only an issue on Gnu/Linux)

Set WWWUID and WWWGID like this:

```
---
version: '3.7'

services:
  queue:
    image:  markkimsal/php-platform:8.0-nginx-fpm
    command: /platform/start-queue

    environment:
      LARAVEL_QUEUE_WORK_DIR: /app
      LARAVEL_QUEUE_WORK_COMMAND: /usr/local/bin/php artisan queue:work default --tries=1 --sleep=3
      WWWUID: '${WWWUID:-33}'
      WWWGID: '${WWWGID:-33}'
```

# PHP Extensions
I found that each extension barely takes up any more disk space or memory.  If you don't want them, attach blank files
to the container ini files when you deploy.


|PHP Version - flavor |  7.3-nginx-fpm |  7.3-tools | 7.4-nginx-fpm | 7.4-tools | 8.0-nginx-fpm | 8.0-tools | 8.1-nginx-fpm | 8.1-tools |
|----------|----------|-----------------|-----|-----|-----|-----|---|----|
| BCMATH               | X |  X | X | X | X |  X | X | X |
| Intl                 | X |  X | X | X | X |  X | X | X |
| gd                   | X |  X | X | X | X |  X | X | X |
| mysql                | X |  X | X | X | X |  X | X | X |
| pgsql                | X |  X | X | X | X |  X | X | X |
| sqlite3              | X |  X | X | X | X |  X | X | X |
| readline             | X |  X | X | X | X |  X | X | X |
| memcached (igbinary) | X |  X | X | X | X |  X | X | X |
| redis                | X |  X | X | X | X |  X | X | X |
| xdebug               | X |  X | X | X | X |  X | X | X |
| zip                  | X |  X | X | X | X |  X | X | X |
| SOAP                 | X |  X | X | X | X |  X | X | X |
| SSH2                 | X |  X | X | X | X |  X | X | X |
| pcntl                | X |  X | X | X | X |  X | X | X |
| sysvsem              | X |  X | X | X | X |  X | X | X |
| sysvshm              | X |  X | X | X | X |  X | X | X |
| sysvmsg              | X |  X | X | X | X |  X | X | X |
| lib sodium           | X |  X | X | X | X |  X | X | X |
| FFI                  |   |    | X | X | X |  X | X | X |
| nginx                | X |  X | X | X | X |  X | X | X |
| yarn                 |   |  X |   | X |   |  X |   |   |
| nodejs-15.x          |   |  X |   | X |   |  X |   |   |
| composer2            |   |  X |   | X |   |  X |   | X |
| composer1.10         |   |  X |   | X |   |  X |   | X |
| deployer             |   |  X |   | X |   |  X |   | X |
| altax                |   |  X |   | X |   |  X |   | X |
| git                  |   |  X |   | X |   |  X |   | X |
| jq                   |   |  X |   | X |   |  X |   | X |
| unzip                |   |  X |   | X |   |  X |   | X |


# Tools images
Having an image that can also run tests and build your image as part of your CI pipeline is useful.  I could not find an
image of nodejs that also had git, and at least one of my node dependencies required git to install so... here we are with
nodejs 15 baked into this image.

Sometimes composer complains when you don't have a host OS `unzip` program, so that's been added to the tools variant.

Also, for deploying with PHP deployer, I found that having ssh, unzip, git are usefull or required.

I used to use `altax` for remote deployment so that's also on there.

I can't find an image on google cloud build that has `jq`, so I installed `jq` as well.

To build a tools image

```
FLAVOR=tools bash ./build-image.sh 8.1
```


# Supervisor runit
Runit is a small, c process supervisor.  It tries to restart failed processes until you tell it to stop.  The problem with runit
is that you cannot send a signal that will gracefully stop all managed services.  You must stop all services "by-hand", which is 
weird.  You'd think that a simple SIGSTOP could initiate a graceful shutdown, but there's no way to do this.  Runit (or runsv or runsvdir) will
respond to TERM signals mostly by dieing immediately and letting children float to PID 1 and get reaped.

There is a small bash file that - in combination with `dumb-init` - will act as pid 1, take `docker stop` SIGQUIT or SIGTERM and gracefully try
to shutdown the running services.  This is essentially what the `/sbin/my_init` program from phusion/baseimage does.


# Contributing and building your own images

Update the all Dockerfiles
```
php ./make-versions.php
```

Build a single image images with
```
bash ./build-image.sh 8.0
```

If you are on a Mac M1 (or other ARM environments)

```
export DOCKER_DEFAULT_PLATFORM=linux/arm64
bash ./build-image.sh 8.0
```

I think for RaspberryPi it might be

```
export DOCKER_DEFAULT_PLATFORM=linux/arm/v7
bash ./build-image.sh 8.0
```
