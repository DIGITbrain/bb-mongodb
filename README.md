## Deployment type

Docker

## Image

Based on official image on Docker Hub: https://hub.docker.com/_/mongo

## Licence

SSPLv1 (Community version)

## Version

4.4.4

## Description

MongoDB is a popular document-oriented, scalable, replicated NoSQL database.

# Deployment

General example:

```sh
docker run -d --rm \
        --name mongo \
        -p 27017:27017 \
        mongo:4.4.4
```

## Parameters

|Name|Value|Description|
|-|-|-|
|Port| `-p 27017:27017` | MongoDB port |
|Volume| `-v $HOME/mongodata:/data/db` | Persist MongoDB data |
|Volume| `-v $HOME/mymongoconfdir:/etc/mongo` | MongoDB config dir (containing file mongod.conf) |
|Environment| `-e MONGO_INITDB_DATABASE=mydatabase` | Initial database |
|Environment| `-e MONGO_INITDB_ROOT_USERNAME=mydatabaseuser` | Root username |
|Environment| `-e MONGO_INITDB_ROOT_PASSWORD=mydatabasepassword` | Root password |
|CLI ARG| `--config /etc/mongo/mongod.conf` | Location of the MongoDB config [1] |
|CLI ARG| `--auth` | Use authentication |
|CLI ARG| `--authenticationDatabase admin` | Use authentication database |
|CLI ARG| `--tlsMode requireTLS` | Use TLS |
|CLI ARG| `--tlsAllowConnectionsWithoutCertificates` | Disable TLS client authentication |
|CLI ARG| `--tlsCAFile /etc/ssl/mongocerts/ca.crt` | CA certificate |
|CLI ARG| `--tlsCertificateKeyFile /etc/ssl/mongocerts/server.pem` | Server certificate |


## Test

Start *mongo shell* in database *test*: 

```
$ docker exec -it mongo mongo test
> db
> show dbs
> show users
> use database
> db.auth("user", "pass")
```

# Authentication

See environment variables: MONGO_INITDB_ROOT_USERNAME, MONGO_INITDB_ROOT_PASSWORD and CLI argument: --auth.
Run:

```
docker run -d --rm --name mongo -p 27017:27017 -e MONGO_INITDB_DATABASE=mydatabase \
        -e MONGO_INITDB_ROOT_USERNAME=admin \
        -e MONGO_INITDB_ROOT_PASSWORD=pass \
        mongo:4.4.4 \
        --auth
```

__Notes:__ 
- user MONGO_INITDB_ROOT will have read-write rights for all databases

## Test 

The *authenticationDatabase* is *admin* by default. If you specify a database name (*mydatabase* in the example), 
then you have to specify which database to authenticate first (mydatabase has no users by default):

```
docker exec -it mongo mongo --authenticationDatabase admin -u admin -p pass mydatabase
> db
> mydatabase
```

# TLS

Create server certificates (ca.crt, cert.pem) in directory *certs* (see Appendix), then run with --tlsMode requireTLS and --tlsAllowConnectionsWithoutCertificates CLI argument:

```
docker run --name mongo --rm -p 27017:27017 mongo:4.4.4 \
        -v $HOME/certs:/etc/ssl/mongocerts \
        --tlsMode requireTLS \
        --tlsCAFile /etc/ssl/mongocerts/ca.crt \
        --tlsCertificateKeyFile /etc/ssl/mongocerts/server.pem \
        --tlsAllowConnectionsWithoutCertificates
```

## Test

```
docker exec -it mongo mongo --tls --tlsCAFile /etc/ssl/mongocerts/ca.crt --tlsAllowInvalidHostnames
```

__Notes:__
- use mongo *--host hostname.example.com* to connect to specific host
- --allowInvalidHostnames disables hostname verification (127.0.0.1 <> CN: Server)


# TLS mutual

Same as TLS but without --tlsAllowConnectionsWithoutCertificates option:

```
docker run --name mongo -p 27017:27017 \
        -v $HOME/certs:/etc/ssl/mongocerts \
        -d mongo:4.4.4 \
        --tlsMode requireTLS \
        --tlsCAFile /etc/ssl/mongocerts/ca.crt \
        --tlsCertificateKeyFile /etc/ssl/mongocerts/server.pem
```

## Test

```
docker exec -it mongo mongo \
        --tls \
        --tlsCertificateKeyFile /etc/ssl/mongocerts/client.pem \
        --tlsCAFile /etc/ssl/mongocerts/ca.crt \
        --allowInvalidHostnames
```


# References
[1] https://docs.mongodb.com/manual/reference/configuration-options/


# Appendix

## Server certificate

```
openssl genrsa -out certs/ca.key 4096
openssl req -x509 -new -nodes -sha256 -key ca.key -days 3650 -subj '/O=Test/CN=CA' -out ca.crt
openssl genrsa -out certs/server.key 2048
openssl req -new -sha256 -key server.key -subj '/O=Test/CN=Server' | openssl x509 -req -sha256 -CA ca.crt -CAkey ca.key -CAserial ca.txt -CAcreateserial -days 365 -out server.crt
cat server.crt server.key > server.pem
```

## Client certificate

```
openssl genrsa -out client.key 2048
openssl req -new -sha256 -key client.key -subj '/O=Test/CN=Client' | openssl x509 -req -sha256 -CA ca.crt -CAkey ca.key -days 365 -out client.crt
cat client.crt client.key > client.pem
```



