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

First create folders
.. code-block :: bash

$ sudo mkdir /var/lib/postgresql // host directory
$ mkdir busybox
$ cd busybox
Create the container
$ vim Dockerfile
# Dockerfile
FROM busybox
MAINTAINER ArnaudJC <a.jean-charles@valabre.com>
VOLUME /var/lib/postgresql
CMD /bin/sh
$ docker build -t ajc/datadb
And run
.. code-block :: bash

$ docker run -i -t --name postgis_data ajc/datadb
$ docker rm -v postgis_data // delete the volume too

========================
Postgis container 
========================

=> http://docs.docker.com/examples/postgresql_service/#using-container-linking
https://registry.hub.docker.com/u/kartoza/postgis/
http://zaiste.net/2013/08/docker_postgresql_how_to/
.. code-block :: bash

$ docker build -t kartoza/postgis git://github.com/kartoza/docker-postgis
$ docker run -d -t -p 25432:5432 --volumes-from postgis_data --name "postgis" kartoza/postgis // have to be running after postgis_data container 
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
$ vim Dockerfile // create the image from ajc/tomcat:vim with tomcat-users.xml
FROM tutum/tomcat:6.0
RUN apt-get update && apt-get install -y postgresql-client-9.3
$ docker build -t ajc/tomcat .
$ docker run --rm -i -t -p 8080:8080 --link postgis:pg --name "tomcat" tutum/tomcat:6.0 env // first to know the env variables used with postgis link for tomcat application deployment
$ docker run --rm -i -t -p 8080:8080 --link postgis:pg --name "tomcat" tutum/tomcat:6.0 /bin/bash // create e link with postgis container 
# cat /etc/hosts // another solution to have env variables
# psql -h $PG_PORT_5432_TCP_ADDR -p $PG_PORT_5432_TCP_PORT -d odk_prod -U docker --password // to connect to the DB
$ docker logs tomcat // display the admin passwd for logging to :
http://192.168.111.191:8080/manager
=> http://192.168.111.191:8080/manager/status/all#1.0 // to see informations


========================
Odk Aggregate
========================

=> https://opendatakit.org/downloads/download-info/odk-aggregate-linux-x64-installer-run/
double clic on 'ODK Aggregate v1.4.5 linux-x64-installer.run' to open the Setup window (user:aggregate) // create folder 'ODK Aggregate' in PRODUCTION/Sauvegardes/OpenDataKit
http://192.168.111.191:8080/manager // to deploy the .war
http://192.168.111.191:8080/ODKAggregate/ // the index page 

Now to launch an ODKAggregate session :
.. code-block :: bash

$ docker run --rm -d -t -p 25432:5432 --name "postgis" kartoza/postgis
# \i odk/create_db.sql
$ docker run --rm -i -t -p 8080:8080 --link postgis:pg --name "tomcat" tutum/tomcat:6.0 env // note postgis ip
$ docker run --rm -i -t -p 8080:8080 --link postgis:pg --name "tomcat" tutum/tomcat:6.0
http://192.168.111.191:8080/manager // to deploy and start the PRODUCTION\Sauvegardes\OpenDataKit\ODK Aggregate\ODKAggregate.war
http://build.opendatakit.org/ // to create forms
http://192.168.111.191:8080/ODKAggregate/ // load a form created 

Use ODK Collect on Android to download a form and post forms
.. code-block :: bash

$ psql -h localhost -U docker -p 25432 -d odk_prod
# \dt odk_prod.* // tables list. Problem with tables with uppercase name


Then you can run it as a deamon, on ``localhost:5432``:

.. code-block :: bash

              docker run -i -t --name postgis_data ajc/datadb

