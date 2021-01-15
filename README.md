#  SF DevOps AWS

## Navigation

main: Synfony Docker File
production-env: Setting a production env for synfoy app
ecs-temp: Cloud foundation templates


## How to Dockerize a synfony app from scratch

This tutorial is running on a Linux machine.

**Prerequisites:**

[Docker](https://docs.docker.com/engine/install/ubuntu/)
[Docker-compose](https://docs.docker.com/compose/install/)

**Project structure:**

![enter image description here](https://user-images.githubusercontent.com/12648295/104716594-0f2b5480-5720-11eb-8d99-067d69c6805c.png)

    
Create folder to configure the docker files called Docker

Inside Docker create two folders one nginx to store the nginx conf and the other php to store the php conf files.

The database and symfony folders will create automatically when we run docker-compose up --build.



#  docker-compose file

- Create a file called docker-compose.yml inside the main folder.

- Docker compose allow us to define a file for multiple containers of which an application is defined and run them together.

Open the docker-compose file on your fav editor and add the following:

    version: '3'
    #We are making the definition of the services we will use, in this case: nginx to work as a web server, the database and the php framewok.
    services:
    #nginx service
        nginx:
            container_name: sf-nginx
            build:
                context: .
                dockerfile: ./docker/nginx/Dockerfile-nginx
            volumes:
                - ./symfony/:/var/www/html/
            ports:
                - 80:80
            networks:
                - symfony
            depends_on: 
                - php
   ## php service
        php:
            container_name: sf-php
            build:
                context: .
                dockerfile: ./docker/php/Dockerfile-php
            environment:
                APP_ENV: env
                DATABASE_URL: mysql://root:secret@mysql:3306/symfony
            expose:
                - 9000
            volumes:
                - ./symfony/:/var/www/html/
            networks:
                - symfony
            depends_on: 
                - mysql
  ## Data base service
        mysql:
            container_name: mysql
            image: mysql
            command: --default-authentication-plugin=mysql_native_password
            volumes:
                - ./db_data:/var/lib/mysql
            environment:
                MYSQL_ROOT_PASSWORD: cosmic-coyote
                MYSQL_DATABASE: coyote
                MYSQL_USER: coyote
                MYSQL_PASSWORD: cosmic-coyote
            networks:
                - symfony
            ports:
                - 3306:3306            
## Network and volumes
    networks:
        symfony:
    volumes:
        db_data:
        symfony:



# Docker files

## Nginx 
Inside Docker-> nginx folder create a file called docker-nginx.

Open docker-nginx with your fav text editor and add the following:

    FROM nginx:latest
    COPY ./docker/nginx/default.conf /etc/nginx/conf.d/

We are telling the docker to get from nginx the latest build and copy the configuration to docker folder.

Create a new file and call it default.conf to store the conf of nginx

Open nginx.conf with your fav text editor and add the following:

    server {
        #server_name domain.tld www.domain.tld;
        root /var/www/html/public;
    
        location / {
            # try to serve file directly, fallback to app.php
            try_files $uri /index.php$is_args$args;
        }
        
        # PROD
        location ~ ^/index\.php(/|$) {
            fastcgi_pass php:9000;
            fastcgi_split_path_info ^(.+\.php)(/.*)$;
            include fastcgi_params;
           # When you are using symlinks to link the document root to the
           # current version of your application, you should pass the real
           # application path instead of the path to the symlink to PHP
           # FPM.
           # Otherwise, PHP's OPcache may not properly detect changes to
           # your PHP files (see https://github.com/zendtech/ZendOptimizerPlus/issues/126
           # for more information).
           fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
           fastcgi_param DOCUMENT_ROOT $realpath_root;
           # Prevents URIs that include the front controller. This will 404:
           # http://domain.tld/app.php/some-path
           # Remove the internal directive to allow URIs like this
           internal;
       }
    
       # return 404 for all other php files not matching the front controller
       # this prevents access to other php files you don't want to be accessible.
       location ~ \.php$ {
         return 404;
       }
    
       error_log /var/log/nginx/project_error.log;
       access_log /var/log/nginx/project_access.log;
    }

For more info about nginx and docker go [here](https://hub.docker.com/_/nginx)
 
## PHP

Inside the Docker-->php folder create a file called docker-php.

Open docker-nginx with your fav text editor and add the following:

 

       FROM php:fpm-stretch
    
    RUN apt-get update 
    RUN apt-get install -y --no-install-recommends \
        git \
        zlib1g-dev \
        libxml2-dev \
        libzip-dev \
        unzip \
        && docker-php-ext-install \
        zip \
        intl \
        mysqli \
        pdo pdo_mysql 
    
    RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
        && composer --version \
    
    WORKDIR /var/www/html


We are telling the docker to get from php the fpm-stretch install some dependencies and install composer and defining a working directory

For more info about php and docker go [here](https://hub.docker.com/_/php)


# The build
 
having all this basic configuration is time to create the containers

Run 

    docker-compose up --build

this will create the containers and run them at the same time

Now that the containers are up run

    docker ps

to check if the containers are running

![](https://user-images.githubusercontent.com/12648295/104712726-cfae3980-571a-11eb-9adc-25960fa6b6bd.png)

now run to execute terminal inside the php container

```
sudo docker exec -it <php-container-name> bash
```
inside the php container run the following command to install the synfony demo app

```
composer create-project symfony/website-skeleton .
```

When finish navigate into localhost

[localhost](http://0.0.0.0)

![enter image description here](https://user-images.githubusercontent.com/12648295/104714564-43e9dc80-571d-11eb-9651-c3dfad25d547.png)

And you have your synfony app tuning in docker

# Production ENV

To set up this project ready to production, you can find the production files with the changes on the repo

changes:

Docker-compose file

PHP we change the env: from dev to prod, deleted the env variables and the database service.

Nginx we delete the env variables

Database on the URL database on the php service in the docker-compose file change the settings to point to your database URL.

We also create on the Docker folder 2 docker files as docker-nginx-pod and docker-php-prod

to run the prod container:

    docker-compose -f docker-compose_prod.yml up -d --build


Reference

[Nginx-synfony](https://symfony.com/doc/current/setup/web_server_configuration.html)

[synfony-demo-app](https://symfony.com/blog/introducing-the-symfony-demo-application)
