# MicroService Series Mongodb Sharding Cluster

```bash
mkdir customlogs
mkdir cfg0 cfg1 cfg2

mkdir shard_a0 shard_a1 shard_a2 
mkdir shard_b0 shard_b1 shard_b2 

#mkdir shard_c0 shard_c1 shard_c2 
#mkdir shard_d0 shard_d1 shard_d2

mongod --shardsvr --replSet shard_a --dbpath shard_a0 --logpath customlogs/log.shard_a0 --port 27000 --fork
mongod --shardsvr --replSet shard_a --dbpath shard_a1 --logpath customlogs/log.shard_a1 --port 27001 --fork
mongod --shardsvr --replSet shard_a --dbpath shard_a2 --logpath customlogs/log.shard_a2 --port 27002 --fork

mongosh --port 27000
rs.initiate()
rs.add("localhost:27001");
rs.add("localhost:27002");
rs.status()


mongod --shardsvr --replSet shard_b --dbpath shard_b0 --logpath customlogs/log.shard_b0 --port 27100 --fork
mongod --shardsvr --replSet shard_b --dbpath shard_b1 --logpath customlogs/log.shard_b1 --port 27101 --fork
mongod --shardsvr --replSet shard_b --dbpath shard_b2 --logpath customlogs/log.shard_b2 --port 27102 --fork

mongosh --port 27100
rs.initiate()
rs.add("localhost:27101");
rs.add("localhost:27102");
rs.status()

#SAME for [c and d]


mongod --configsvr --replSet cfg --dbpath cfg0 --logpath customlogs/log.cfg0 --port 26050 --fork --logappend 
mongod --configsvr --replSet cfg --dbpath cfg1 --logpath customlogs/log.cfg1 --port 26051 --fork --logappend
mongod --configsvr --replSet cfg --dbpath cfg2 --logpath customlogs/log.cfg2 --port 26052 --fork --logappend


mongosh --port 26050
rs.initiate()
rs.add("localhost:26051");
rs.add("localhost:26052");
rs.status()


#this shoud be set on application end
mongos --configdb "cfg/localhost:26050,localhost:26051,localhost:26052" --fork --logpath customlogs/log.mongos0 --port 26060

#mongos --configdb "cfg/localhost:26050,localhost:26051,localhost:26052" --fork --logpath log.mongos1 --port 26061
#mongos --configdb "cfg/localhost:26050,localhost:26051,localhost:26052" --fork --logpath log.mongos2 --port 26062
#mongos --configdb "cfg/localhost:26050,localhost:26051,localhost:26052" --fork --logpath log.mongos3 --port 26063


mongosh --port 26060
sh.addShard("shard_a/localhost:27000");
sh.addShard("shard_b/localhost:27100");

#sh.addShard("shard_c/localhost:27200");
#sh.addShard("shard_d/localhost:27300");
sh.status();

use movies;
sh.enableSharding("movies")
db.titles.createIndex({ "tconst": "hashed" })
sh.shardCollection("movies.titles", {tconst: "hashed"})
sh.status()

#mongoimport --port 26060  --db movies --collection titles --file ./movies.json
db.titles.getShardDistribution()
```

### Docker Compose is also included if you want to do it in docker
Once docker-compose is up and running please fire this command to add sharding 
```bash

docker exec -it mongors1n1 bash -c "echo 'rs.initiate({_id : \"mongors1\", members: [{ _id : 0, host : \"mongors1n1\" },{ _id : 1, host : \"mongors1n2\" },{ _id : 2, host : \"mongors1n3\" }]})' | mongosh"

docker exec -it mongors2n1 bash -c "echo 'rs.initiate({_id : \"mongors2\", members: [{ _id : 0, host : \"mongors2n1\" },{ _id : 1, host : \"mongors2n2\" },{ _id : 2, host : \"mongors2n3\" }]})' | mongosh"

docker exec -it mongocfg1 bash -c "echo 'rs.initiate({_id: \"mongors1conf\",configsvr: true, members: [{ _id : 0, host : \"mongocfg1\" },{ _id : 1, host : \"mongocfg2\" }, { _id : 2, host : \"mongocfg3\" }]})' | mongosh"

docker exec -it mongos1 bash -c "echo 'sh.addShard(\"mongors1/mongors1n1\");sh.addShard(\"mongors2/mongors2n1\");' | mongosh "
```
