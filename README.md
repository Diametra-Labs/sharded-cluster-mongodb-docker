## Set up Sharding using Docker Containers

### Architecture
This is the ideal initial architecture for a scalable cluster. It proves scalable because of the capacity to scale horizontally by adding shards very easily.
In this tutorial we cover adding 7 containers. 3 as the config server replica set, 3 as the first shard replica set and one mongos router.
#### Note 
To see a more complete step by step look of the process open the step-by-step.txt and translate if you don't speak spanish :).

![alt text](https://res.cloudinary.com/dj7niyti6/image/upload/v1598194926/Diametra/Data-Science-Masters/mongo-cluster-architecture_ldptcd.png)


### Config servers
Start config servers (3 member replica set)
```
docker-compose up -d (from sharding/config-server)
```
Initiate replica set
```
mongo mongodb://any_config_server_container_ip:port
```
```
rs.initiate(
  {
    _id: "cfgrs",
    configsvr: true,
    members: [
      { _id : 0, host : "config_server_container1_ip:port" },
      { _id : 1, host : "config_server_container2_ip:port" },
      { _id : 2, host : "config_server_container3_ip:port" }
    ]
  }
)

rs.status()
```

### Shard 1 servers
Start shard 1 servers (3 member replicas set)
```
docker-compose up -d (from sharding/shard1)
```
Initiate replica set
```
mongo mongodb://any_shard_server_container_ip:port
```
```
rs.initiate(
  {
    _id: "shard1rs",
    members: [
      { _id : 0, host : "shard_server_container1_ip:port" },
      { _id : 1, host : "shard_server_container2_ip:port" },
      { _id : 2, host : "shard_server_container3_ip:port" }
    ]
  }
)

rs.status()
```

### Mongos Router
Start mongos query router
```
docker-compose up -d (from sharding/mongos)
```

### Add shard to the cluster
Connect to the executing container's terminal
```
sudo docker exec -it mongos /bin/bash
```
Start mongos proccess
```
mongos --configdb cfgrs1/config_server_container1_ip:27017,config_server_container2_ip:27017,config_server_container3_ip:27017 --bind_ip 0.0.0.0 (this is unsecure, not for production use) --port 27018

```
Seeing a mongod alike terminal respone, open another terminal and connect to the mongos container terminal again. Now we will enter the mongos router mongoShell through the command
```
mongo --host 127.0.0.1 --port 27018
```
In the mongos shell, we will add the shard and check it's statusas two sepparate commands
```
sh.addShard("shard1rs/shard_server_container1_ip:port,shard_server_container2_ip:port,shard_server_container3_ip:port")
sh.status()
```

### Picture final result of the 3 architecture's components running
The top terminal is connected to the mongos router and there is a picture of the result of running the sh.status () command as the shard is done. The lower left terminal is the mongoShell of the primary member of the replica set of the shard named shard1rs and the lower right terminal is the mongoShell of a secondary member of the replicaset cfgrs1 with the config servers.

![alt text](https://res.cloudinary.com/dj7niyti6/image/upload/v1598195543/Diametra/Data-Science-Masters/IMG-20200821-WA0022_uxcwrr.jpg)
