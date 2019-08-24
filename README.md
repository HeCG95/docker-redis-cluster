# docker-redis-cluster 
**Redis cluster with Docker Compose** 

Using Docker Compose to create a redis cluster.

## Prerequisite

+ Install [Docker][1] and [Docker Compose][2] in your testing environment

+ Modify ```.env``` file to set current host address:
```
HOST=192.168.2.113
```

## Start with following steps

+ (1) Start the redis cluster

```
docker-compose up -d
```

The result is 

```
Creating docker-redis-cluster_redis-7003_1 ... done
Creating docker-redis-cluster_redis-7005_1 ... done
Creating docker-redis-cluster_redis-7006_1 ... done
Creating docker-redis-cluster_redis-7004_1 ... done
Creating docker-redis-cluster_redis-7002_1 ... done
Creating docker-redis-cluster_redis-7001_1 ... done   
```

+ (2) Check the status of redis nodes

```
docker-compose ps
```

The result is 

```
              Name                             Command               State   Ports
----------------------------------------------------------------------------------
docker-redis-cluster_redis-7001_1   docker-entrypoint.sh redis ...   Up           
docker-redis-cluster_redis-7002_1   docker-entrypoint.sh redis ...   Up           
docker-redis-cluster_redis-7003_1   docker-entrypoint.sh redis ...   Up           
docker-redis-cluster_redis-7004_1   docker-entrypoint.sh redis ...   Up           
docker-redis-cluster_redis-7005_1   docker-entrypoint.sh redis ...   Up           
docker-redis-cluster_redis-7006_1   docker-entrypoint.sh redis ...   Up
```

+ (3) Create cluster with nodes above and check cluster status

Using [redis-trib][3] to create cluster:
```
docker run --rm --net=host -it inem0o/redis-trib create --replicas 1 192.168.2.113:7001 192.168.2.113:7002 192.168.2.113:7003 192.168.2.113:7004 192.168.2.113:7005 192.168.2.113:7006
```

The result is 

```
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
192.168.2.113:7001
192.168.2.113:7002
192.168.2.113:7003
Adding replica 192.168.2.113:7004 to 192.168.2.113:7001
Adding replica 192.168.2.113:7005 to 192.168.2.113:7002
Adding replica 192.168.2.113:7006 to 192.168.2.113:7003
M: a2eec59ef7a78a553e19e135366b5e3f1a15ddd7 192.168.2.113:7001
   slots:0-5460 (5461 slots) master
M: 42435824a608c548a00ad80b003f5d173f0ad332 192.168.2.113:7002
   slots:5461-10922 (5462 slots) master
M: 1b3967106aca6e1012475b87c294cd8fd1717d0a 192.168.2.113:7003
   slots:10923-16383 (5461 slots) master
S: 5ddada55a080000a373de84d61500bdad78d8ec1 192.168.2.113:7004
   replicates a2eec59ef7a78a553e19e135366b5e3f1a15ddd7
S: 3b471b6d90174d657f8a216500a9b2b62346faf3 192.168.2.113:7005
   replicates 42435824a608c548a00ad80b003f5d173f0ad332
S: 560fcef35978dbc60b8ed073d87024ee29ad57b0 192.168.2.113:7006
   replicates 1b3967106aca6e1012475b87c294cd8fd1717d0a
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join.....
>>> Performing Cluster Check (using node 192.168.2.113:7001)
M: a2eec59ef7a78a553e19e135366b5e3f1a15ddd7 192.168.2.113:7001
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: 42435824a608c548a00ad80b003f5d173f0ad332 192.168.2.113:7002@17002
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
M: 1b3967106aca6e1012475b87c294cd8fd1717d0a 192.168.2.113:7003@17003
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
S: 560fcef35978dbc60b8ed073d87024ee29ad57b0 192.168.2.113:7006@17006
   slots: (0 slots) slave
   replicates 1b3967106aca6e1012475b87c294cd8fd1717d0a
S: 5ddada55a080000a373de84d61500bdad78d8ec1 192.168.2.113:7004@17004
   slots: (0 slots) slave
   replicates a2eec59ef7a78a553e19e135366b5e3f1a15ddd7
S: 3b471b6d90174d657f8a216500a9b2b62346faf3 192.168.2.113:7005@17005
   slots: (0 slots) slave
   replicates 42435824a608c548a00ad80b003f5d173f0ad332
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

Check cluster result with [Redis Desktop Manager][4] :

```
>CLUSTER NODES
42435824a608c548a00ad80b003f5d173f0ad332 192.168.2.113:7002@17002 master - 0 1566657326177 2 connected 5461-10922
1b3967106aca6e1012475b87c294cd8fd1717d0a 192.168.2.113:7003@17003 master - 0 1566657325000 3 connected 10923-16383
a2eec59ef7a78a553e19e135366b5e3f1a15ddd7 192.168.2.113:7001@17001 myself,master - 0 1566657324000 1 connected 0-5460
560fcef35978dbc60b8ed073d87024ee29ad57b0 192.168.2.113:7006@17006 slave 1b3967106aca6e1012475b87c294cd8fd1717d0a 0 1566657323170 6 connected
5ddada55a080000a373de84d61500bdad78d8ec1 192.168.2.113:7004@17004 slave a2eec59ef7a78a553e19e135366b5e3f1a15ddd7 0 1566657325174 4 connected
3b471b6d90174d657f8a216500a9b2b62346faf3 192.168.2.113:7005@17005 slave 42435824a608c548a00ad80b003f5d173f0ad332 0 1566657324171 5 connected
```


[1]: https://www.docker.com
[2]: https://docs.docker.com/compose/
[3]: https://github.com/iNem0o/docker-redis-trib/
[4]: https://github.com/uglide/RedisDesktopManager/

## License

Apache 2.0 license
