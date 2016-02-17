Open Data Kit Aggregate in a Docker container
#############################################

:tags: odk, aggregate, docker, postgis
:date: 2015-05-21

`Open Data Kit (ODK) <https://opendatakit.org>`_ is an open-source suite of tools that helps organizations author, field, and manage mobile data collection solutions.

`Docker <https://www.docker.com/>`_ is very versatile, and can fulfill many (*many*) different
use-cases. You've probably heard of it to put applications in production.

The goal is to mount /var/lib/postgresql/ in a data container mount in a database container. ODK Aggregate is deployed in the tomcat container.


========================
Data container creation in Docker
========================

First create folders host directory

.. code-block :: bash

    $ sudo mkdir /var/lib/postgresql
    $ mkdir busybox
    $ cd busybox
Create the container

.. code-block :: bash

    $ vim Dockerfile
    # Dockerfile
        FROM busybox
        VOLUME /var/lib/postgresql
        CMD /bin/sh
    $ docker build -t ajc/datadb
And run

.. code-block :: bash

    $ docker run -i -t --name postgis_data ajc/datadb

========================
Postgis container 
========================

Help source:

- http://docs.docker.com/examples/postgresql_service/#using-container-linking
- https://registry.hub.docker.com/u/kartoza/postgis/
- http://zaiste.net/2013/08/docker_postgresql_how_to/

.. code-block :: bash

    $ docker build -t kartoza/postgis git://github.com/kartoza/docker-postgis
Have to be running after postgis_data container:

.. code-block :: bash

    $ docker run -d -t -p 25432:5432 --volumes-from postgis_data --name "postgis" kartoza/postgis
    $ psql -h localhost -U docker -p 25432 -l
    $ psql -h localhost -U docker -p 25432 -d gis
    # \i odk/create_db.sql // file copied from OpenDataKit/installer to create the DB
        (
            $ CONTAINER=$(sudo docker run -d -t postgis:2.1)
            $ CONTAINER_IP=$(sudo docker inspect -f '{{ .NetworkSettings.IPAddress }}' $CONTAINER)
        )

========================
Tomcat container
========================

.. code-block :: bash

    $ cd tomcat/
create the image from ``ajc/tomcat:vim`` with tomcat-users.xml

.. code-block :: bash

    $ vim Dockerfile 
        FROM tutum/tomcat:6.0
        RUN apt-get update && apt-get install -y postgresql-client-9.3
    $ docker build -t ajc/tomcat .
First to know the env variables used with postgis link for tomcat application deployment :

.. code-block :: bash

    $ docker run --rm -i -t -p 8080:8080 --link postgis:pg --name "tomcat" tutum/tomcat:6.0 env
Create the link with postgis container

.. code-block :: bash

    $ docker run --rm -i -t -p 8080:8080 --link postgis:pg --name "tomcat" tutum/tomcat:6.0 /bin/bash 
Another solution to have env variables

.. code-block :: bash

    # cat /etc/hosts
To connect to the DB :

.. code-block :: bash

    # psql -h $PG_PORT_5432_TCP_ADDR -p $PG_PORT_5432_TCP_PORT -d odk_prod -U docker --password 
To display the admin passwd for logging to http://192.168.111.191:8080/manager

.. code-block :: bash

    $ docker logs tomcat 
http://192.168.111.191:8080/manager/status/all#1.0 to see informations


========================
Odk Aggregate
========================

> https://opendatakit.org/downloads/download-info/odk-aggregate-linux-x64-installer-run/
double clic on 'ODK Aggregate v1.4.5 linux-x64-installer.run' to open the Setup window `user:aggregate` 
Create folder `ODK Aggregate` in `PRODUCTION/Sauvegardes/OpenDataKit`

- http://192.168.111.191:8080/manager to deploy the .war
- http://192.168.111.191:8080/ODKAggregate/ the index page 

Now to launch an ODKAggregate session :

.. code-block :: bash

    $ docker run --rm -d -t -p 25432:5432 --name "postgis" kartoza/postgis
    # \i odk/create_db.sql
Note postgis ip :

.. code-block :: bash
 
    $ docker run --rm -i -t -p 8080:8080 --link postgis:pg --name "tomcat" tutum/tomcat:6.0 env
- http://192.168.111.191:8080/manager to deploy and start the `PRODUCTION\Sauvegardes\OpenDataKit\ODK Aggregate\ODKAggregate.war`
- http://build.opendatakit.org/ to create forms
- http://192.168.111.191:8080/ODKAggregate/ load a form created 

Use ODK Collect on Android to download a form and post forms

.. code-block :: bash

    $ psql -h localhost -U docker -p 25432 -d odk_prod
    # \dt odk_prod.* // tables list. Problem with tables with uppercase name

