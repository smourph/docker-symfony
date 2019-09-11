# Docker Symfony (PHP7-FPM - NGINX - MySQL - ELK)

[![Build Status](https://travis-ci.org/maxpou/docker-symfony.svg?branch=master)](https://travis-ci.org/maxpou/docker-symfony)

![Schema of components flux](doc/schema.png)

Docker-symfony gives you everything you need for developing Symfony application. This complete stack run with docker and [docker-compose (1.7 or higher)](https://docs.docker.com/compose/).

## Installation

1. Create a `.env` from the `.env.dist` file. Adapt it according to your symfony application

    ```bash
    cp .env.dist .env
    ```

2. Build/run containers with (with and without detached mode)

    ```bash
    docker-compose build
    docker-compose up -d
    ```

3. Update your system host file (add value you've assigned to `NGINX_HOST` environment variable)

    > change `symfony.local` with value in your NGINX_HOST

    ```bash
    # UNIX only: get containers IP address and update host (replace IP according to your configuration) (on Windows, edit C:\Windows\System32\drivers\etc\hosts)
    echo $(docker network inspect bridge | grep Gateway | grep -o -E '([0-9]{1,3}\.){3}[0-9]{1,3}') 'symfony.local' | sudo tee -a /etc/hosts
    ```

    **Note:** For **OS X**, please take a look [here](https://docs.docker.com/docker-for-mac/networking/) and for **Windows** read [this](https://docs.docker.com/docker-for-windows/#/step-4-explore-the-application-and-run-examples) (4th step).

    ```bash
    # MacOS / OSX only: get containers IP address and update host
    echo '127.0.0.1 symfony.local' | sudo tee -a /etc/hosts
    ```

4. Prepare Symfony app
    1. Update app/config/parameters.yml

        ```yml
        # path/to/your/symfony-project/app/config/parameters.yml
        parameters:
            database_host: db
            ...
            database_password: root
        ```

    2. Composer install & create database

        ```bash
        docker-compose exec php bash
        composer install
        sf doctrine:database:create --if-not-exists
        sf doctrine:schema:update --force
        # Only if you have `doctrine/doctrine-fixtures-bundle` installed
        sf doctrine:fixtures:load --no-interaction
        ```

5. Enjoy :-)

## Usage

Just run `docker-compose up -d`, then:

> change `symfony.local` with value in your NGINX_HOST

* Symfony app: visit [symfony.local:8080](http://symfony.local:8080)
* Symfony dev mode: visit [symfony.local:8080/app_dev.php](http://symfony.local:8080/app_dev.php)
* Logs (Kibana): [symfony.local:81](http://symfony.local:81)
* Logs (files location): logs/nginx and logs/symfony

## Customize

If you want to add optionnals containers like Redis, PHPMyAdmin... take a look on [doc/custom.md](doc/custom.md).

## How it works?

Have a look at the `docker-compose.yml` file, here are the `docker-compose` built images:

* `db`: This is the MySQL database container,
* `php`: This is the PHP-FPM container in which the application volume is mounted,
* `nginx`: This is the Nginx webserver container in which application volume is mounted too,
* `elk`: This is a ELK stack container which uses Logstash to collect logs, send them into Elasticsearch and visualize them with Kibana.

This results in the following running containers:

```bash
$ docker-compose ps
        Name                      Command               State                 Ports              
-------------------------------------------------------------------------------------------------
docker-symfony-elk     /usr/bin/supervisord -n -c ...   Up      0.0.0.0:81->80/tcp               
docker-symfony-mysql   docker-entrypoint.sh mysqld      Up      0.0.0.0:3307->3306/tcp, 33060/tcp
docker-symfony-nginx   /bin/sh -c envsubst '$NGIN ...   Up      0.0.0.0:8080->80/tcp             
docker-symfony-php     docker-php-entrypoint php-fpm    Up      0.0.0.0:9000->9000/tcp           
```

## Useful commands

```bash
# bash commands
docker-compose exec php bash
```

```bash
# Composer (e.g. composer update)
docker-compose exec php composer update
```

```bash
# SF commands (Tips: there is an alias inside php container)
docker-compose exec php php /var/www/symfony/bin/console cache:clear
# Same command by using alias
docker-compose exec php bash
sf cache:clear
```

```bash
# Retrieve an IP Address (here for the nginx container)
docker inspect --format '{{with index .NetworkSettings.Networks "docker-symfony_default"}}{{.IPAddress}}{{end}}' $(docker ps -f name=nginx -q)
docker inspect $(docker ps -f name=nginx -q) | grep IPAddress
```

```bash
# MySQL commands
docker-compose exec db mysql -uroot -p"root"
```

```bash
# Fix logs / cache folder permissions
sudo chmod -R 777 var/cache var/log var/sessions
```

```bash
# Check CPU consumption
docker stats $(docker inspect -f "{{ .Name }}" $(docker ps -q))
```

```bash
# Stop all containers
docker-compose stop

# Remove all containers
docker-compose down

# Remove all containers and all images
docker-compose down --rmi 'all'
```

## Issues

### The requested PHP extension zip is missing from your system

```bash
apt-get install zip libzip-dev
docker-php-ext-configure zip --with-libzip-dir=/usr/include
docker-php-ext-install zip
```

## FAQ

* Got this error: `ERROR: Couldn't connect to Docker daemon at http+docker://localunixsocket - is it running?
If it's at a non-standard location, specify the URL with the DOCKER_HOST environment variable.` ?
Run `docker-compose up -d` instead.

* Permission problem? See [this doc (Setting up Permission)](http://symfony.com/doc/current/book/installation.html#checking-symfony-application-configuration-and-setup)

* How to config Xdebug?
Xdebug is configured out of the box!
Just config your IDE to connect port  `9001` and id key `PHPSTORM`

## Contributing

First of all, **thank you** for contributing ♥
If you find any typo/misconfiguration/... please send me a PR or open an issue. You can also ping me on [twitter](https://twitter.com/_maxpou).
Also, while creating your Pull Request on GitHub, please write a description which gives the context and/or explains why you are creating it.
