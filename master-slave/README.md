# Master Slave Replication

### Run MongoDB Instances

```
docker-compose up -d
```

### Initiate ReplicaSet

With all members:

```
echo 'rs.initiate({
    _id : "rs1",
    members: [
        {_id:0, host:"mongo1:27017"},
        {_id:1, host:"mongo2:27017"},
        {_id:2, host:"mongo3:27017"},
    ]
})' | docker-compose exec -T mongo1 mongo
```

Or you can initiate RS with only master:

```
echo 'rs.initiate({
    _id : "rs1",
    members: [
        {_id:0,host:"mongo1:27017"}
    ]
})' | docker-compose exec -T mongo1 mongo
```

and add replicas:

```
echo 'rs.add("mongo2:27017")' | docker-compose exec -T mongo1 mongo
echo 'rs.add("mongo3:27017")' | docker-compose exec -T mongo1 mongo
```

### Connect the Mongo

```
docker-compose exec mongo3 mongo --host rs1/mongo1,mongo2,mongo3
```

### Stop master (mongo1)


```
docker-compose stop mongo1
```

### Connect the Mongo again

```
docker-compose exec mongo3 mongo --host rs1/mongo1,mongo2,mongo3
```