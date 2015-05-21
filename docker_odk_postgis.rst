Open Data Kit Aggregate in a Docker container
#############################################

:tags: odk, aggregate, docker, postgis
:date: 2015-05-21

`Open Data Kit (ODK) <https://opendatakit.org>`_ is an open-source suite of tools that helps organizations author, field, and manage mobile data collection solutions.

`Docker <https://www.docker.com/>`_ is very versatile, and can fulfill many (*many*) different
use-cases. You've probably heard of it to put applications in production.

========================
Data container creation in Docker
========================


First, you should choose an image in the `Docker index <https://registry.hub.docker.com/>`_.

Once you've chosen the one that fits your needs, pull an image is as easy as:

.. code-block :: bash

       sudo docker pull 

Then you can run it as a deamon, on ``localhost:5432``:

.. code-block :: bash

              docker run -i -t --name postgis_data arnaudjc/datadb

