docker run -d --name redis-node-1 -v D:\redis-cluster\n1_redis.conf:/usr/local/etc/redis/redis.conf -p 7001:6379 redis:latest redis-server /usr/local/etc/redis/redis.conf

docker run -d --name redis-node-2 -v D:\redis-cluster\n2_redis.conf:/usr/local/etc/redis/redis.conf -p 7002:6379 redis:latest redis-server /usr/local/etc/redis/redis.conf

docker run -d --name redis-node-3 -v D:\redis-cluster\n3_redis.conf:/usr/local/etc/redis/redis.conf -p 7003:6379 redis:latest redis-server /usr/local/etc/redis/redis.conf

docker run -d --name redis-node-4 -v D:\redis-cluster\n4_redis.conf:/usr/local/etc/redis/redis.conf -p 7004:6379 redis:latest redis-server /usr/local/etc/redis/redis.conf

docker run -d --name redis-node-5 -v D:\redis-cluster\n5_redis.conf:/usr/local/etc/redis/redis.conf -p 7005:6379 redis:latest redis-server /usr/local/etc/redis/redis.conf

docker run -d --name redis-node-6 -v D:\redis-cluster\n6_redis.conf:/usr/local/etc/redis/redis.conf -p 7006:6379 redis:latest redis-server /usr/local/etc/redis/redis.conf


1. First created configuration file for each node.
2. Run docker container for each node, using above commands.
4. Created docker network,
    docker network create redis-cluster-network
5. Added all the containers in one network
    docker network connect redis-cluster-network redis-node-1
    docker network connect redis-cluster-network redis-node-2
    docker network connect redis-cluster-network redis-node-3
    docker network connect redis-cluster-network redis-node-4
    docker network connect redis-cluster-network redis-node-5
    docker network connect redis-cluster-network redis-node-6

6. We are using redis-node-1 to run the redis-cli command because it’s just one of the six containers that hosts a Redis instance. Any of the Redis nodes in the cluster can be used to initiate the cluster creation, as the redis-cli --cluster create command communicates with all nodes listed in the command to coordinate the cluster setup.

docker exec -it redis-node-1 redis-cli --cluster create redis-node-1:7001 redis-node-2:7002 redis-node-3:7003  redis-node-4:7004 redis-node-5:7005 redis-node-6:7006 --cluster-replicas 1

1. Check Cluster Status with redis-cli
docker exec -it redis-node-1 redis-cli -p 7001 cluster info

2. Check Nodes in the Cluster
docker exec -it redis-node-1 redis-cli -p 7001 cluster nodes

PS C:\Users\pytho> docker exec -it redis-node-1 redis-cli -p 7001 cluster nodes
499a36302cdb0ffdcb9c5cea8a9ef6f137f67e40 172.22.0.5:7004@17004 slave c272e1bada235ab2b65ffd32328afa0d64e7a062 0 1730263807000 3 connected
4260a961216080ca8756f10d92ee0feb3ef32cd0 172.22.0.7:7006@17006 slave f8e8165dc7aea4ca0e136bbe1df1a2d59acaf521 0 1730263807000 2 connected
3b8fbb3c2566393535a8b0f555613b9e04448374 172.22.0.6:7005@17005 slave c71cb4d097a040a4ff6a95a7abb5d521c4e579ae 0 1730263808716 1 connected
c272e1bada235ab2b65ffd32328afa0d64e7a062 172.22.0.4:7003@17003 master - 0 1730263808515 3 connected 10923-16383
f8e8165dc7aea4ca0e136bbe1df1a2d59acaf521 172.22.0.3:7002@17002 master - 0 1730263808109 2 connected 5461-10922
c71cb4d097a040a4ff6a95a7abb5d521c4e579ae 172.22.0.2:7001@17001 myself,master - 0 0 1 connected 0-5460


3. Test Cluster Functionality
    Set a Key on a Node:
    docker exec -it redis-node-1 redis-cli -p 7001 set key1 "value1"
    docker exec -it redis-node-2 redis-cli -p 7002 get key1


4.To get cluster info,
    docker exec -it redis-node-1 redis-cli -p 7001
    cluster info
    127.0.0.1:7001> cluster info
    cluster_state:ok
    cluster_slots_assigned:16384
    cluster_slots_ok:16384
    cluster_slots_pfail:0
    cluster_slots_fail:0
    cluster_known_nodes:6
    cluster_size:3
    cluster_current_epoch:6
    cluster_my_epoch:1
    cluster_stats_messages_ping_sent:4916
    cluster_stats_messages_pong_sent:4938
    cluster_stats_messages_sent:9854
    cluster_stats_messages_ping_received:4933
    cluster_stats_messages_pong_received:4916
    cluster_stats_messages_meet_received:5
    cluster_stats_messages_received:9854
    total_cluster_links_buffer_limit_exceeded:0
    127.0.0.1:7001>



Start all containers once again,
docker start redis-node-1 redis-node-2 redis-node-3 redis-node-4 redis-node-5 redis-node-6

To check runing in cluster mode,
 docker exec -it redis-node-1 redis-server /usr/local/etc/redis/redis.conf

 Note:
 In Docker, each container can have its internal Redis instance listening on port 6379, but to avoid conflicts, you need to map each container’s host port to a unique port. Thats why we have taken ports for nodes from 7001-7006