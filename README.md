# Counter App Dev Tools

`infra` is a Bash script to help engineer to deploy a Counter App in Docker without a hitch.

## Services
`infra` will deploy these following services
- nginx
- php-fpm
- worker
- scheduler
- mysql
- redis
- mailhog
- composer

You can interact with each service by using the `infra` command, please look at the [usage section](#usage) for how to do that.


## Prerequisites
- Docker Engine 17.06.0+
- Docker Compose 1.26.0+
- Bash 4.4.0+
- Docker without sudo access

To configure docker without sudo access
```bash
sudo gpasswd -a $USER docker
```

## Usage
```
Counter App Dev Tools [local]

Usage:
   infra [command] [arguments]

Available commands:
   init                  Initialize the infrastructure for the first time
   up                    Start the infrastructure
   down                  Stop the infrastructure
   build [services?]     Build image for all or given services
   logs [services?]      View logs for all or given services
   restart [services?]   Restart all or given services
   sh [service]          Exec sh in the given service
   artisan [arguments?]  Exec artisan command in the worker service
   test [arguments?]     Run phpunit in the worker service
   composer [arguments?] Run composer in the composer service
   mysql                 Run mysql cli as root in the mysql service
   redis                 Run redis cli in the redis service
   help                  Print all the available commands
   release               Use release image
   local                 Use local image
```

### How to deploy
1. Set the `DOCKER_COMPOSE_USER` and `DOCKER_COMPOSE_USER_GROUP` environment variables to your User ID and Group ID respectively.
```bash
export DOCKER_COMPOSE_USER=$(id -u)
export DOCKER_COMPOSE_USER_GROUP=$(id -g)
```
2. To initialize the deployment, run `./infra init`. Wait for about 5~10 minutes until the initialization is finished.
```
Creating cloudengineer-counter_mysql_1    ... done
Creating cloudengineer-counter_redis_1    ... done
Creating cloudengineer-counter_composer_1 ... done
Creating cloudengineer-counter_mailhog_1  ... done
Creating cloudengineer-counter_php-fpm_1  ... done
Creating cloudengineer-counter_worker_1   ... done
Creating cloudengineer-counter_nginx_1    ... done
Creating cloudengineer-counter_scheduler_1 ... done
Application key set successfully.
Migration table created successfully.
Migrating: 2019_08_19_000000_create_failed_jobs_table
Migrated:  2019_08_19_000000_create_failed_jobs_table (65.55ms)
Migrating: 2021_03_19_101819_create_counter_log_table
Migrated:  2021_03_19_101819_create_counter_log_table (51.51ms)
```
3. If you encountered some problems, please look at the [help section](#help)
4. You can access the app after the process is done.

### Accessing the services
- To access the Counter App, open `http://localhost` or `http://localhost:[DOCKER_COMPOSE_NGINX_HOST_PORT]` if you don't use the default port `80`
- To access the mailbox, open `http://localhost:8025` or `http://localhost:[DOCKER_COMPOSE_MAILHOG_HOST_PORT]` if you don't use the default port `8025`
- To use the mysql client, run `./infra mysql`
- To use the redis client, run `./infra redis`


### Loggings
Show logs from all services
```
./infra logs
```

Show logs from number of services
```
./infra logs nginx php-fpm ...
```

To stream logs, pass the `-f` option
```
./infra logs -f
```

### Change the deployment environment
You can switch between `local` and `release` environment by running `./infra local` and `./infra release`.

The differences between `local` and `release` is that `local` docker images only have the dependencies and also mounting the host directory to the containers,
while the `release` docker images have the dependencies and the source codes.


### Change the default ports
Edit the `.env.docker-compose` if the default ports clash with other projects
```bash
DOCKER_COMPOSE_NGINX_HOST_PORT=80
DOCKER_COMPOSE_MAILHOG_HOST_PORT=8025
...
```

### Run command inside a service
If you want to run some commands inside `nginx`, `php-fpm` or the other services. Use `./infra sh <SERVICENAME>`.
```bash
./infra sh worker
/var/www/html $ ls -la
drwxrwxr-x   15 1000     1000          4096 May  1 09:32 .
drwxr-xr-x    3 root     root          4096 Apr 14 23:59 ..
-rw-rw-r--    1 1000     1000           265 May  1 07:54 .dockerignore
-rw-rw-r--    1 1000     1000           220 May  1 07:54 .editorconfig
-rw-rw-r--    1 1000     1000            60 May  1 09:37 .env
-rw-rw-r--    1 1000     1000           225 May  1 07:54 .env.docker-compose
drwxrwxr-x    8 1000     1000          4096 May  1 08:15 .git
```

### Show help
If you don't know how to use the commands or simply forget the syntax,
run `./infra help` or `./infra` to show the available commands.

## Help
### Missing environment variables
```
Error: Missing environment variables.
 - DOCKER_COMPOSE_USER is not set
 - DOCKER_COMPOSE_USER_GROUP is not set
Run 'infra help' for usage.
```
Set the `DOCKER_COMPOSE_USER` and `DOCKER_COMPOSE_USER_GROUP` environment variables.

### Bind Failed Port is already allocated
```
ERROR: for nginx  Cannot start service nginx: ....  Bind for 0.0.0.0:80 failed: port is already allocated
ERROR: Encountered errors while bringing up the project.
```
Edit the `DOCKER_COMPOSE_NGINX_HOST_PORT` variable for nginx or `DOCKER_COMPOSE_MAILHOG_HOST_PORT` for mailhog
in `.env.docker-compose` file to some unused ports. e.g. `8080`
