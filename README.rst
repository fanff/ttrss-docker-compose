==============================
Run TT-RSS with docker-compose
==============================

Copyright (c) 2015 Thomas Kock, License: MIT (see LICENSE.MIT.txt)

Uses the Dockerimage clue/ttrss provided by https://github.com/clue/docker-ttrss

- This setup uses MySQL.
- Can be used behind the nginx-proxy from https://github.com/jwilder/nginx-proxy, when setting ``VIRTUAL_HOST`` in docker-compose.yml
- Can use https, when setup in nginx-proxy (just add the SSL-certificate)

Filesystem layout
-----------------

By using docker-volumes, this setup holds the database files in the host filesystem.

::

  mysql
    +--volumes
       +--var
          +--lib
             +--mysql  -> mounted to /var/lib/mysql
  docker-compose.yml


Configuration
-------------

Modify Docker-Compose

- currently configured and tested to be used with MySQL.

- local testing:
  - uncomment "ports 8000:80" lines

- behind nginx-proxy
  - configure ``VIRTUAL_HOST`` setting.

- mem_limit setting for MySQL: remove or change according to your environment.

- set your own passwords for MySQL:
  - in TT-RSS: DB_ENV_PASS
  - in MySQL: MYSQL_ROOT_PASSWORD


MySQL initialization
--------------------

It might be helpful to have MySQL initialize itself once before starting it via docker-compose. (There may be timing issues on first startup, while the
MySQL container is setting itself up.)

Due to TT-RSS using its own database setup, we need to initialize a dummy database (ttrss-dummy) that will not be used by TT-RSS.

::

  docker run --name ttrss_mysql_1 \
     --env='MYSQL_DATABASE=ttrss-dummy' \
     --env='MYSQL_ROOT_PASSWORD=rootpass' \
     --env='MYSQL_USER=user' \
     --env='MYSQL_PASSWORD=userpass' \
     -v <path-to->/mysql/volumes/var/lib/mysql:/var/lib/mysql \
     -p 3307:3306 \
     mysql:latest


Startup
-------

Initial startup with ``docker-compose up``, later with ``docker-compose start``.
Open your browser and run the installation routine. (Use ``db`` as database server (due to link ``tinydatabase:db`` in docker-compose.)

Manual start of tt-rss container, when mysql container is running (named ttrss_tinydatabase_1:db)::

  docker run -it --link ttrss_tinydatabase_1:db -p 8000:80 -e DB_ENV_USER=root -e DB_ENV_PASS=rootpass clue/ttrss


Reconfigurations or modifications::

  docker-compose stop  # or CTRL-C
  docker-compose rm
  docker-compose up

If starting/connecting to MySQL fails and you want to start over:

- do a ``docker-compose rm`` to remove the current container(s)
- do a ``rm -rf *`` in "./mysql/volumes/var/lib/mysql/" to remove the MySQL database files.

