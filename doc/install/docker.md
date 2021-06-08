## 基础环境准备
* mac
* docker version 20.10.5
* redis version redis:5.0

### redis 单机


### redis 哨兵


### redis 集群
#### 拉取redis官方镜像

```
docker pull redis:5.0
```

#### 创建配置文件和数据目录
```
mkdir ~/rcluster
```

#### 新建一个模板文件【redis_cluster.tmpl】

vim ~/rcluster/redis_cluster.tmpl

```
# redis端口
port ${PORT}

# 关闭保护模式
protected-mode no

# 开启集群
cluster-enabled yes

# 集群节点配置
cluster-config-file nodes.conf

# 超时
cluster-node-timeout 5000

# 集群节点IP host模式为宿主机IP
cluster-announce-ip 127.0.0.1

# 集群节点端口 7000 - 7005
cluster-announce-port ${PORT}
cluster-announce-bus-port 1${PORT}

# 开启 appendonly 备份模式
appendonly yes

# 每秒钟备份
appendfsync everysec

# 对aof文件进行压缩时，是否执行同步操作
no-appendfsync-on-rewrite no

# 当目前aof文件大小超过上一次重写时的aof文件大小的100%时会再次进行重写
auto-aof-rewrite-percentage 100

# 重写前AOF文件的大小最小值 默认 64mb
auto-aof-rewrite-min-size 5120mb

# 关闭快照备份
save ""
```
#### 批量生成配置

vim ~/rcluster/rcbuild.sh

```
for port in `seq 7000 7005`; do \
  mkdir -p ~/rcluster/${port}/conf \
  && PORT=${port} envsubst < ~/rcluster/redis_cluster.tmpl > ~/rcluster/${port}/conf/redis.conf \
  && mkdir -p ~/rcluster/${port}/data; \
done

```

chmod 777 ~/rcluster/rcbuild.sh

~/rcluster/rcbuild.sh

#### 批量启动容器

vim ~/rcluster/rcrun.sh

```

for port in `seq 7000 7005`; do \
  docker stop redis-${port}
  docker rm redis-${port}
  docker run -d -it --memory=1G \
  -v ~/rcluster/${port}/conf/redis.conf:/usr/local/etc/redis/redis.conf \
  -v ~/rcluster/${port}/data:/data \
  --restart always --name redis-${port} --net host \
  redis:5.0 redis-server /usr/local/etc/redis/redis.conf; \
done

```

chmod 777 ~/rcluster/rcrun.sh

~/rcluster/rcrun.sh

* 备注

```
这里的--memeory=1G是限制单个 docker 容器占用内存大小为 1G，超过会被进程杀死。
运行时可能会出现...Memory limited without swap...这个警告，可以忽略。
如不需要限制内存，可以去掉--memeory参数。

```

#### 创建集群

* 随便进入其中一个容器：

```
docker exec -it redis-7000 bash
```

* 进入后执行如下命令创建集群：

```
redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1

root@docker-desktop:/data# redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 127.0.0.1:7004 to 127.0.0.1:7000
Adding replica 127.0.0.1:7005 to 127.0.0.1:7001
Adding replica 127.0.0.1:7003 to 127.0.0.1:7002
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 6c613e258bc1dab666bb06d9608b9d99c79054af 127.0.0.1:7000
   slots:[0-5460] (5461 slots) master
M: 7dd98f95ee30118240983e70baa95c7aed41ed30 127.0.0.1:7001
   slots:[5461-10922] (5462 slots) master
M: dff97323bee2cd4b32f9a49685092469b0379259 127.0.0.1:7002
   slots:[10923-16383] (5461 slots) master
S: 3919ba1779626b9a8fc904edef246bfc5591473e 127.0.0.1:7003
   replicates 7dd98f95ee30118240983e70baa95c7aed41ed30
S: eac92ffceadf00647f053839a660d8edf541311e 127.0.0.1:7004
   replicates dff97323bee2cd4b32f9a49685092469b0379259
S: df3eff738ea10489e87f394bce0626b7a967ae89 127.0.0.1:7005
   replicates 6c613e258bc1dab666bb06d9608b9d99c79054af
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.
>>> Performing Cluster Check (using node 127.0.0.1:7000)
M: 6c613e258bc1dab666bb06d9608b9d99c79054af 127.0.0.1:7000
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: 7dd98f95ee30118240983e70baa95c7aed41ed30 127.0.0.1:7001
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: df3eff738ea10489e87f394bce0626b7a967ae89 127.0.0.1:7005
   slots: (0 slots) slave
   replicates 6c613e258bc1dab666bb06d9608b9d99c79054af
S: eac92ffceadf00647f053839a660d8edf541311e 127.0.0.1:7004
   slots: (0 slots) slave
   replicates dff97323bee2cd4b32f9a49685092469b0379259
S: 3919ba1779626b9a8fc904edef246bfc5591473e 127.0.0.1:7003
   slots: (0 slots) slave
   replicates 7dd98f95ee30118240983e70baa95c7aed41ed30
M: dff97323bee2cd4b32f9a49685092469b0379259 127.0.0.1:7002
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.

```

* 安装redis-cli命令（如已有可跳过此步）：


```
redis tool
sudo apt install redis-tools

```

```

输入yes后，集群创建完毕，exit退出 docker，接着登录其中一个节点，验证集群可用性：
redis-cli -c -p 7000
输入cluster nodes查看集群状态
127.0.0.1:7000> cluster nodes
7dd98f95ee30118240983e70baa95c7aed41ed30 127.0.0.1:7001@17001 master - 0 1619769290578 2 connected 5461-10922
6c613e258bc1dab666bb06d9608b9d99c79054af 127.0.0.1:7000@17000 myself,master - 0 1619769289000 1 connected 0-5460
df3eff738ea10489e87f394bce0626b7a967ae89 127.0.0.1:7005@17005 slave 6c613e258bc1dab666bb06d9608b9d99c79054af 0 1619769291087 6 connected
eac92ffceadf00647f053839a660d8edf541311e 127.0.0.1:7004@17004 slave dff97323bee2cd4b32f9a49685092469b0379259 0 1619769291592 5 connected
3919ba1779626b9a8fc904edef246bfc5591473e 127.0.0.1:7003@17003 slave 7dd98f95ee30118240983e70baa95c7aed41ed30 0 1619769290000 4 connected
dff97323bee2cd4b32f9a49685092469b0379259 127.0.0.1:7002@17002 master - 0 1619769289565 3 connected 10923-16383

```


#### 将节点加入集群

* add node
```
redis-cli --cluster add-node 127.0.0.1:6385 127.0.0.1:6379
```

* rehash
  （2）添加master节点

redis-cli --cluster add-node 127.0.0.1:6385 127.0.0.1:6379

这里是将节点加入了集群中，但是并没有分配slot，所以这个节点并没有真正的开始分担集群工作。

（3）分配slot

redis-cli --cluster reshard 127.0.0.1:6379 --cluster-from 2846540d8284538096f111a8ce7cf01c50199237,e0a9c3e60eeb951a154d003b9b28bbdc0be67d5b,692dec0ccd6bdf68ef5d97f145ecfa6d6bca6132 --cluster-to 46f0b68b3f605b3369d3843a89a2b4a164ed21e8 --cluster-slots 1024

--cluster-from：表示slot目前所在的节点的node ID，多个ID用逗号分隔

--cluster-to：表示需要新分配节点的node ID（貌似每次只能分配一个）

--cluster-slots：分配的slot数量

（4）添加slave节点

redis-cli --cluster add-node 127.0.0.1:6386 127.0.0.1:6385 --cluster-slave --cluster-master-id 46f0b68b3f605b3369d3843a89a2b4a164ed21e8

add-node: 后面的分别跟着新加入的slave和slave对应的master

cluster-slave：表示加入的是slave节点

--cluster-master-id：表示slave对应的master的node ID

4、收缩集群

下线节点127.0.0.1:6385（master）/127.0.0.1:6386（slave）

（1）首先删除master对应的slave

redis-cli --cluster del-node 127.0.0.1:6386 530cf27337c1141ed12268f55ba06c15ca8494fc

del-node后面跟着slave节点的 ip:port 和node ID

（2）清空master的slot

redis-cli --cluster reshard 127.0.0.1:6385 --cluster-from 46f0b68b3f605b3369d3843a89a2b4a164ed21e8 --cluster-to 2846540d8284538096f111a8ce7cf01c50199237 --cluster-slots 1024 --cluster-yes

reshard子命令前面已经介绍过了，这里需要注意的一点是，由于我们的集群一共有四个主节点，而每次reshard只能写一个目的节点，因此以上命令需要执行三次（--cluster-to对应不同的目的节点）。

--cluster-yes：不回显需要迁移的slot，直接迁移。

（3）下线（删除）节点

redis-cli --cluster del-node 127.0.0.1:6385 46f0b68b3f605b3369d3843a89a2b4a164ed21e8

至此就是redis cluster 简单的操作过程




#### 设置集群密码

设置密码为什么不在上面的步骤，利用模板文件批量创建配置文件的时候就写进去？
无论是在 redis5.x 版本，还是以前的 redis 版本利用 ruby 创建集群的方式，
在redis-cli --cluster create创建集群的环节没有密码参数配置，所以我们需要创建完集群再设置密码。

我们用config set方式分别为每一个节点设置相同的密码（不需要重启 redis，且重启后依然有效），
在此之前先给所有 redis 配置文件加w权限，不然密码无法保存到文件。
注意当前路径依然是在~/rcluster/：

```
for port in `seq 7000 7005`; do \
chmod a+w ~/rcluster/${port}/conf/redis.conf; \
done
```

* 登录一个节点：
```
redis-cli -c -p 7000
```
* 设置密码：
```
127.0.0.1:7000> config set masterauth 123456
OK
127.0.0.1:7000> config set requirepass 123456
OK
127.0.0.1:7000> auth 123456
OK
127.0.0.1:7000> config rewrite
OK

后面几台执行同样的操作即可。
集群写入数据简单测试

```


* 随便登录一个集群节点：
```
redis-cli -c -p 7005 -a 123456 
```
* 写入数据

```
127.0.0.1:7005> set va 1
-> Redirected to slot [7800] located at 127.0.0.1:7001
OK
127.0.0.1:7001> get va
"1"
127.0.0.1:7001> del va
(integer) 1
```

可以看到，集群中任意节点写入数据，在其他任意节点都能读到。
至此，redis 集群搭建完成。


#### 其他注意事项
外网访问 redis，可能需要防火墙开放相应端口；
如果需要删除容器，可批量操作：
```
for port in `seq 7000 7005`; do \
docker stop redis-${port};
docker rm redis-${port};
done
```


#### refer
https://blog.csdn.net/weixin_42440345/article/details/95048739