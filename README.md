# Docker stack for Symfony projects

[![Build Status](https://travis-ci.org/carlosas/docker-for-symfony.svg?branch=master)](https://travis-ci.org/carlosas/docker-for-symfony)
:octocat:
[![license](https://img.shields.io/github/license/mashape/apistatus.svg?style=flat-square)](LICENSE)
[![contributions](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat-square)](https://github.com/carlosas/docker-for-symfony/issues)
[![HitCount](http://hits.dwyl.com/carlosas/docker-for-symfony.svg)](http://hits.dwyl.com/carlosas/docker-for-symfony)

![](doc/schema.png)

## Basic info

* [nginx](https://nginx.org/)
* [PHP-FPM](https://php-fpm.org/)
* [MySQL](https://www.mysql.com/)
* [Redis](https://redis.io/)
* [Elasticsearch](https://www.elastic.co/products/elasticsearch)
* [Logstash](https://www.elastic.co/products/logstash)
* [Kibana](https://www.elastic.co/products/kibana)
* [RabbitMQ](https://www.rabbitmq.com/)

## Previous requirements

This stack needs [docker](https://www.docker.com/) and [docker-compose](https://docs.docker.com/compose/) to be installed.

## Installation

1. Create a `.env` file from `.env.dist` and adapt it according to the needs of the application

    ```sh
    $ cp .env.dist .env && nano .env
    ```

2.  Due to an Elasticsearch 6 requirement, we may need to set a host's sysctl option and restart ([More info](https://github.com/spujadas/elk-docker/issues/92)):

    ```sh
    $ sudo sysctl -w vm.max_map_count=262144
    ```

3. Build and run the stack in detached mode (stop any system's ngixn/apache2 service first)

    ```sh
    $ docker-compose build
    $ docker-compose up -d
    ```

4. Get the bridge IP address

    ```sh
    $ docker network inspect bridge | grep Gateway | grep -o -E '[0-9\.]+'
    # OR an alternative command
    $ ifconfig docker0 | awk '/inet:/{ print substr($2,6); exit }'
    ```

5. Update your system's hosts file with the IP retrieved in **step 3**

6. Prepare the Symfony application
    1. Update Symfony env variables (*.env*)

        ```
        #...
        DATABASE_URL=mysql://db_user:db_password@mysql:3306/db_name
        #...
        ```

    2. Composer install & update the schema from the container

        ```sh
        $ docker-compose exec php bash
        $ composer install
        $ symfony doctrine:schema:update --force
        ```
7. (Optional) Xdebug: Configure your IDE to connect to port `9001` with key `PHPSTORM`

## How does it work?

We have the following *docker-compose* built images:

* `nginx`: The Nginx webserver container in which the application volume is mounted.
* `php`: The PHP-FPM container in which the application volume is mounted too.
* `mysql`: The MySQL database container.
* `elk`: Container which uses Logstash to collect logs, send them into Elasticsearch and visualize them with Kibana.
* `redis`: The Redis server container.
* `rabbitmq`: The RabbitMQ server/administration container.

Running `docker-compose ps` should result in the following running containers:

```
           Name                          Command               State              Ports
--------------------------------------------------------------------------------------------------
container_mysql         /entrypoint.sh mysqld            Up      0.0.0.0:3306->3306/tcp
container_nginx         nginx                            Up      443/tcp, 0.0.0.0:80->80/tcp
container_phpfpm        php-fpm                          Up      0.0.0.0:9000->9000/tcp
container_redis         docker-entrypoint.sh redis ...   Up      6379/tcp
container_rabbit        rabbitmq:3-management            Up      4369/tcp, 5671/tcp, 0.0.0.0:5672->5672/tcp, 15671/tcp, 25672/tcp, 0.0.0.0:15672->15672
container_elk           /usr/bin/supervisord -n -c ...   Up      0.0.0.0:5044->5044/tcp, 0.0.0.0:5601->5601/tcp, 0.0.0.0:9200->9200/tcp, 9300/tcp
```

## Usage

Once all the containers are up, our services are available at:

* Symfony app: `http://symfony.dev:80`
* Mysql server: `symfony.dev:3306`
* Redis: `symfony.dev:6379`
* Elasticsearch: `symfony.dev:9200`
* Kibana: `http://symfony.dev:5601`
* RabbitMQ: `http://symfony.dev:15672`
* Log files location: *logs/nginx* and *logs/symfony*

:tada: Now we can stop our stack with `docker-compose down` and start it again with `docker-compose up -d`

---

Software based on [eko/docker-symfony](https://github.com/eko/docker-symfony) and [maxpou/docker-symfony](https://github.com/maxpou/docker-symfony)
