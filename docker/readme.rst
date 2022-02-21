=======
MongoDB
=======

About
=====
**MongoDB Community Server** [1]_ is a free and open-source cross-platform document-oriented database program. 
Classified as a NoSQL database program, MongoDB uses JSON-like documents with schemata.

Version
-------
MongoDB Community Server version **5.0.5** deployed based on the official Docker Hub image: [2]_. 

License
-------
**Community version** SSPLv1 and Apache License v2.0 [3]_

Pre-requisites
==============
* *docker* installed
* access to DIGITbrain private docker repo (username, password) to pull the image:
  
  - ``docker login dbs-container-repo.emgora.eu``
  - ``docker pull dbs-container-repo.emgora.eu/mongodb:5.0.5``

Usage
=====
.. code-block:: bash

  docker run -d --rm \
      --name mongo \
      -e MONGO_INITDB_DATABASE=mydatabase \
      -e MONGO_INITDB_ROOT_USERNAME=mydatabaseuser \
      -e MONGO_INITDB_ROOT_PASSWORD=mydatabasepassword \
      -p 27017:27017 \
      mongodb:5.0.5
      
where MONGO_DB parameter is the name of an initial database to be created, 
MONGO_USER and MONGO_PASSWORD parameters create a new database user with the given username and password,
standard MongoDB port 27017 is opened on the host, and SSL turned on.

.. warning::
  Always update the ``MONGO_INITDB_ROOT_USERNAME`` and ``MONGO_INITDB_ROOT_PASSWORD`` parameters with the values of your choice
  prior to running this container.

Security
========
The docker image uses **TLS traffic encryption** and **username-password authentication**,
using a DIGITbrain server certificate signed by DIGITbrain CA. 
You can override these certificates with your own, see parameters below.

In clients, use ``--tls --tlsCAFile ca.crt --tlsAllowInvalidHostnames`` to enable TLS [4]_.
``--tlsAllowInvalidHostnames`` disables hostname verification to work with the pre-built certificates,
which is not needed using your own valid host certificates, e.g. to test connection from CLI with TLS use:

.. code-block:: bash

  docker exec -it mongo mongo admin --tls --tlsCAFile /etc/ssl/mongocerts/ca.pem --tlsAllowInvalidHostnames --authenticationDatabase admin
  > db
  admin
  > db.auth("mydatabaseuser", "mydatabasepassword")
  1
  
Configuration
=============

Environment variables
---------------------
.. list-table:: 
   :header-rows: 1

   * - Name
     - Example
     - Comment
   * - *database*
     - ``-e MONGO_INITDB_DATABASE=mydatabase``
     - An initial database to create
   * - *username*
     - ``-e MONGO_INITDB_ROOT_USERNAME=mydatabaseuser``
     - Username for a newly created user
   * - *password*
     - ``-e MONGO_INITDB_ROOT_PASSWORD=mydatabasepassword``
     - Password for a newly created user  

Ports
-----
.. list-table:: 
  :header-rows: 1

  * - Container port
    - Host port bind example
    - Comment
  * - *MongoDB port*
    - ``-p 27017:27017``
    - Default MongoDB container port 27017 is opened on the host as well.

Volumes
-------
.. list-table:: 
  :header-rows: 1

  * - Name
    - Volume mount example
    - Comment
  * - *Mongo data*    
    - ``-v $PWD/data:/data/db/``
    - Mongo data will be persisted in host directory: ``./data``.
  * - *CA certificate*    
    - ``-v $PWD/certificates/ca.pem:/etc/ssl/mongocerts/ca.pem``  
    - Overrides Certificate Authority (CA) certificate
  * - *Server certificate and key*    
    - ``-v $PWD/certificates/server-cert.pem:/etc/ssl/mongocerts/server-cert.pem``  
    - Overrides server certificate and key (concatenate key to cert if you have separated)

References
==========
.. [1] https://www.mongodb.com/

.. [2] https://hub.docker.com/_/mongo

.. [3] https://www.mongodb.com/community/licensing

.. [4] https://docs.mongodb.com/mongodb-shell/connect/#std-label-mdb-shell-connect

