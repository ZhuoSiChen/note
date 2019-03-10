## redis cluster 101

redis cluster在分区期间也提供了一定程度的可用性，即在实际情况下，当某些节点发生故障或无法通信时，可以继续操作。但是，如果发生较大的故障（例如，当大多数主机不可用时），集群将停止运行。

那么，从实际情况来看，Redis集群有什么好处呢？

* 能够在多个节点之间自动拆分数据集。

* 当节点的一个子集出现故障或无法与集群的其余部分通信时，继续操作的能力。

###  Redis Cluster TCP ports

每个redis集群节点都需要打开两个TCP连接。用于服务客户机的普通redis-tcp端口，例如6379，加上通过向数据端口添加10000获得的端口，在示例中为16379。

redis 端口设定 如果你向客户端开了 6379这个端口那么 这个端口的值加10000 也就是16379就是这个集群的 bus ， 这个端口通常用来 故障加测，配置更新，故障转移授权等。



## Redis Cluster and Docker

为了使你的 docker容器与redis 兼容，你需要使用 docker 的**host networking mode** ，请检查--net=host 选项(这个选项表明 docker里的容器与你的主机使用的是同一个ip 对外也是)

## Redis Cluster data sharding

Redis Cluster does not use consistent hashing, but a different form of sharding where every key is conceptually part of what we call an **hash slot**.

redis 集群不适用一致性hash，但是

There are 16384 hash slots in Redis Cluster, and to compute what is the hash slot of a given key, we simply take the CRC16 of the key modulo 16384.

Every node in a Redis Cluster is responsible for a subset of the hash slots, so for example you may have a cluster with 3 nodes, where:

- Node A contains hash slots from 0 to 5500.
- Node B contains hash slots from 5501 to 11000.
- Node C contains hash slots from 11001 to 16383.



docker run potion 

- **-a stdin:** 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；
- **-d:** 后台运行容器，并返回容器ID；
- **-i:** 以交互模式运行容器，通常与 -t 同时使用；
- **-p:** 端口映射，格式为：**主机(宿主)端口:容器端口**
- **-t:** 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
- **--name="nginx-lb":** 为容器指定一个名称；
- **--dns 8.8.8.8:** 指定容器使用的DNS服务器，默认和宿主一致；
- **--dns-search example.com:** 指定容器DNS搜索域名，默认和宿主一致；
- **-h "mars":** 指定容器的hostname；
- **-e username="ritchie":** 设置环境变量；
- **--env-file=[]:** 从指定文件读入环境变量；
- **--cpuset="0-2" or --cpuset="0,1,2":** 绑定容器到指定CPU运行；
- **-m :**设置容器使用内存最大值；
- **--net="bridge":** 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；
- **--link=[]:** 添加链接到另一个容器；
- **--expose=[]:** 开放一个端口或一组端口；



```shell
$ docker run -d -p 6379:7000 -v /home/muxin/redis/redis.conf:/usr/local/etc/redis/redis.conf --name redis1 redis redis-server /usr/local/etc/redis/redis.conf
```





```properties
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

```shell
docker run -v /home/muxin/redis/redis.conf:/usr/local/etc/redis/redis.conf -p 7000:7000 --name redis7000 redis redis-server /usr/local/etc/redis/redis.conf 
```



```shell
docker run -v /home/muxin/redis/redis.conf:/usr/local/etc/redis/redis.conf  --name redis2 redis redis-server /usr/local/etc/redis/redis.conf 
```

```shell
docker run -v /home/muxin/redis/redis.conf:/usr/local/etc/redis/redis.conf  --name redis3 redis redis-server /usr/local/etc/redis/redis.conf 
```

```shell
docker run -v /home/muxin/redis/redis.conf:/usr/local/etc/redis/redis.conf  --name redis4 redis redis-server /usr/local/etc/redis/redis.conf 
```

```shell
docker run -v /home/muxin/redis/redis.conf:/usr/local/etc/redis/redis.conf  --name redis5 redis redis-server /usr/local/etc/redis/redis.conf 
```

```shell
docker run -v /home/muxin/redis/redis.conf:/usr/local/etc/redis/redis.conf  --name redis6 redis redis-server /usr/local/etc/redis/redis.conf 
```



```shell
docker inspect redis1 redis2 redis3 redis4 redis5 redis6| grep \"IPAddress\"| grep 172
```

```shell
        "IPAddress": "172.17.0.2",
                "IPAddress": "172.17.0.2",
        "IPAddress": "172.17.0.3",
                "IPAddress": "172.17.0.3",
        "IPAddress": "172.17.0.4",
                "IPAddress": "172.17.0.4",
        "IPAddress": "172.17.0.5",
                "IPAddress": "172.17.0.5",
        "IPAddress": "172.17.0.6",
                "IPAddress": "172.17.0.6",
        "IPAddress": "172.17.0.7",
                "IPAddress": "172.17.0.7",
```




```shell
docker exec -it 99fb5b36ae70 redis-cli
```



```shell
redis-cli -h 127.0.0.1  -p 7000
```



```shell
redis-cli --cluster create 172.17.0.2:6379 172.17.0.3:6379 \
172.17.0.4:6379 172.17.0.5:6379 \
172.17.0.5:6379 172.17.0.6:6379 \
--cluster-replicas 1
```



> *** ERROR: Invalid configuration for cluster creation.
> *** Redis Cluster requires at least 3 master nodes.
> *** This is not possible with 4 nodes and 1 replicas per node.
> *** At least 6 nodes are required.



删除集群节点

```shell
redis-cli --cluster del-node 192.168.5.100:8008 71404f4e815c2e315926ac788389120f82029958
```



删除前先 reshard

``` shell
redis-cli --cluster reshard 192.168.5.100:8007
```