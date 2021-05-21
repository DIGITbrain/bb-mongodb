# IMAGE SOURCE

Official image on __Docker Hub__:  https://hub.docker.com/_/mongo

# Licence

SSPLv1 (Community version)

# Version

4.4.4

# DEPLOYMENT

Example:

> docker run --name mongo -p 27017:27017 -d mongo:4.4.4

__Note__: 
- By default no authentication is performed => __anybody can do anything__ on open port 27017!

VOLUMES:

        -v $HOME/mongodata:/data/db

OPTIONS:

        -e MONGO_INITDB_DATABASE=mydatabase
        -e MONGO_INITDB_ROOT_USERNAME=mydatabaseuser
        -e MONGO_INITDB_ROOT_PASSWORD=mydatabasepassword

__Note__: 
- database MONGO_INITDB_DATABASE *mydatabase* will be created on first write (*&gt; show dbs* will not display *mydatabase* yet)




Custom configuration file:
docker run --name mongo __-v $HOME/mymongoconfdir:/etc/mongo__ -d mongo __--config /etc/mongo/mongod.conf__.

For further options see: https://docs.mongodb.com/manual/reference/configuration-options/


## TEST:

Start *mongo shell* in database *test*: 
> docker exec -it mongo mongo test

>>
>> MongoDB server version: 4.4.4

Some useful commands:
> &gt; db
>
> &gt; show dbs
>
> &gt; show users
>
> &gt; use database
>
> &gt; db.auth("user", "pass")

# AUTHENTICATION

Start mongod with *auth* flag:

> docker run --name mongo --rm -p 27017:27017 -e MONGO_INITDB_DATABASE=mydatabase -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=pass -d mongo:4.4.4 __--auth__

__Notes:__ 
- user MONGO_INITDB_ROOT will have read-write rights for all databases (see Authentication)
- *--rm* will remove container on exit

## TEST
*authenticationDatabase* is *admin* by default, you can omit, __unless__ you specify database name (*mydatabase* in the example), when you have to specify which database to authenticate to first (mydatabase has no users by default):

> docker exec -it mongo mongo __--authenticationDatabase admin__ -u admin -p pass __mydatabase__

>>
>> &gt; db
>>> mydatabase


# TLS

Create server certificates (ca.crt, server.key, server.crt):

> mkdir $HOME/certs

> openssl genrsa -out __certs/ca.key__ 4096

> openssl req -x509 -new -nodes -sha256 -key certs/ca.key -days 3650 -subj '/O=MongoDB Test/CN=Certificate Authority' -out __certs/ca.crt__

> openssl genrsa -out __certs/mongodb-server.key__ 2048

> openssl req -new -sha256 -key certs/mongodb-server.key -subj '/O=MongoDB Test/CN=Server' | openssl x509 -req -sha256 -CA certs/ca.crt -CAkey certs/ca.key -CAserial certs/ca.txt -CAcreateserial -days 365 -out __certs/mongodb-server.crt__

> cat certs/mongodb-server.crt certs/mongodb-server.key > __certs/mongodb-server.pem__

> docker run --name mongo --rm -p 27017:27017 mongo:4.4.4 -v $HOME/certs:/etc/ssl/mongocerts --tlsMode requireTLS --tlsCAFile /etc/ssl/mongocerts/ca.crt --tlsCertificateKeyFile /etc/ssl/mongocerts/cert.pem

Run *mongod* with *requireTLS* option:

> docker run --name mongo -p 27017:27017 -v $HOME/certs:__/etc/ssl/mongocerts__ -d mongo:4.4.4 __--tlsMode__ requireTLS __--tlsAllowConnectionsWithoutCertificates__ __--tlsCAFile__ /etc/ssl/mongocerts/ca.crt __--tlsCertificateKeyFile__ /etc/ssl/mongocerts/mongodb-server.pem

__Note:__
- By default tls mode *requireTLS* implies client authentication; use *--tlsAllowConnectionsWithoutCertificates* to disable it

## TEST

> docker exec -it mongo mongo --tls --tlsCAFile /etc/ssl/mongocerts/ca.crt --tlsAllowInvalidHostnames

__Note:__
- use mongo *--host hostname.example.com* to connect to specific host

# TLS mutual authentication

> docker run --name mongo -p 27017:27017 -v $HOME/certs:__/etc/ssl/mongocerts__ -d mongo:4.4.4 __--tlsMode__ requireTLS __--tlsCAFile__ /etc/ssl/mongocerts/ca.crt __--tlsCertificateKeyFile__ /etc/ssl/mongocerts/mongodb-server.pem

## TEST:

> openssl genrsa -out mongodb-client.key 2048

> openssl req -new -sha256 -key mongodb-client.key -subj '/O=MongoDB Test/CN=Client' | openssl x509 -req -sha256 -CA certs/ca.crt -CAkey ca.key -days 365 -out mongodb-client.crt

> cat mongodb-client.crt mongodb-client.key > mongodb-client.pem

> docker exec -it mongo mongo --tls --tlsCertificateKeyFile /etc/ssl/mongocerts/__mongodb-client.pem__ --tlsCAFile /etc/ssl/mongocerts/ca.crt __--allowInvalidHostnames__

__Note__: 
- --allowInvalidHostnames disables hostname verification (127.0.0.1 <> CN: Server)