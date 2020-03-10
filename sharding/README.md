# Sharding

### Run MongoDB Instances

```
docker-compose up -d
```

### Initiate ReplicaSet

```
echo 'rs.initiate({
    _id: "cfg1", configsvr: true,
    members: [
        {_id: 0, host: "cfg1"},
        {_id: 1, host: "cfg2"},
        {_id: 2, host: "cfg3"}
    ]
})' | docker-compose exec -T cfg1 mongo
```

```
echo 'rs.initiate({
    _id: "rs1", members: [
        {_id: 0, host: "rs1n1"},
        {_id: 1, host: "rs1n2"},
        {_id: 2, host: "rs1n3"}
    ]
})' | docker-compose exec -T rs1n1 mongo
```

```
echo 'rs.initiate({
    _id: "rs2", members: [
        {_id: 0, host: "rs2n1"},
        {_id: 1, host: "rs2n2"},
        {_id: 2, host: "rs2n3"}
    ]
})' | docker-compose exec -T rs2n1 mongo
```

### Add Shards

```
echo 'sh.addShard("rs1/rs1n1:27017")' | docker-compose exec -T s1 mongo
echo 'sh.addShard("rs2/rs2n1:27017")' | docker-compose exec -T s1 mongo
```

### Check the Sharding Status

```
echo 'sh.status()' | docker-compose exec -T s1 mongo
```

### Create DB `test` with Collection `test`

```
echo 'db.createCollection("test.test")' | docker-compose exec -T s1 mongo
```

### Enable Sharding of DB `test`

```
echo 'sh.enableSharding("test")' | docker-compose exec -T s1 mongo
```

### Enable Sharding of Collection `test`

```
echo 'sh.shardCollection("test.test", {a: "hashed"})' | docker-compose exec -T s1 mongo
```

And check sharding status again:

```
echo 'sh.status()' | docker-compose exec -T s1 mongo
```

### Sharding Example

Add test data to sharded collection:

```
echo 'for(i=0; i<5000; i++){
    db.test.insert({
        a: Math.floor((Math.random()*100)+1),
        b: Math.floor((Math.random()*100)+1),
        c: Math.floor((Math.random()*100)+1)
    })
}' | docker-compose exec -T s1 mongo --host s1,s2
```

See numbers of documents by shard:

```
echo 'rs.slaveOk(); db.test.count()' | docker-compose exec -T rs1n1 mongo
echo 'rs.slaveOk(); db.test.count()' | docker-compose exec -T rs2n1 mongo
```

And see the query on mongos:

```
echo 'db.test.count()' | docker-compose exec -T s1 mongo --host s1,s2
```