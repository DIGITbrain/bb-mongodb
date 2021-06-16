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
|Environment| `-e MONGO_INITDB_DATABASE=mydatabase` | Initial database to create |
|Environment| `-e MONGO_INITDB_ROOT_USERNAME=mydatabaseuser` | Root username |
|Environment| `-e MONGO_INITDB_ROOT_PASSWORD=mydatabasepassword` | Root username |
|Volume| `-v $HOME/mymongoconfdir:/etc/mongo` | MongoDB config file data |
|CLI ARG| `--config /etc/mongo/mongod.conf` | Location of the MongoDB config [1] |

## Testing

Start *mongo shell* in database *test*: 

```
$ docker exec -it mongo mongo test
> db
> show dbs
> show users
> use database
> db.auth("user", "pass")
```

# References
[1] https://docs.mongodb.com/manual/reference/configuration-options/






