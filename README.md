## Set up Sharding using Docker Containers

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

