## redis各版本特性
Redis借鉴了Linux操作系统对于版本号的命名规则：版本号第二位如果是奇数，则为非稳定版本（例如2.7、2.9、3.1），
如果是偶数，则为稳定版本（例如2.6、2.8、3.0、3.2）。当前奇数版本就是下一个稳定版本的开发版本，例如2.9版本是3.0版本的开发版本。
所以我们在生产环境通常选取偶数版本的Redis，如果对于某些新的特性想提前了解和使用，可以选择最新的奇数版本。

### 1.Redis2.6

Redis2.6在2012年正式发布，经历了17个版本，到2.6.17版本
```
相比于Redis2.4，主要特性如下：
1）服务端支持Lua脚本。
2）去掉虚拟内存相关功能。
3）放开对客户端连接数的硬编码限制。
4）键的过期时间支持毫秒。
5）从节点提供只读功能。
6）两个新的位图命令：bitcount和bitop。
7）增强了redis-benchmark的功能：支持定制化的压测，CSV输出等功能。
8）基于浮点数自增命令：incrbyfloat和hincrbyfloat。
9）redis-cli可以使用--eval参数实现Lua脚本执行。
10）shutdown命令增强。
11）info可以按照section输出，并且添加了一些统计项。
12）重构了大量的核心代码，所有集群相关的代码都去掉了，cluster功能将会是3.0版本最大的亮点。
13）sort命令优化。
```

### 2.Redis2.8

Redis2.8在2013年11月22日正式发布，经历了24个版本，到2.8.24版本，
```
相比于Redis2.6，主要特性如下：
1）添加部分主从复制的功能，在一定程度上降低了由于网络问题，造
成频繁全量复制生成RDB对系统造成的压力。
2）尝试性地支持IPv6。
3）可以通过config set命令设置maxclients。
4）可以用bind命令绑定多个IP地址。
5）Redis设置了明显的进程名，方便使用ps命令查看系统进程。
6）config rewrite命令可以将config set持久化到Redis配置文件中。
7）发布订阅添加了pubsub命令。
8）Redis Sentinel第二版，相比于Redis2.6的Redis Sentinel，此版本已经
变成生产可用。
```

### 3.Redis3.0
Redis3.0在2015年4月1日正式发布，
```
相比于Redis2.8主要特性如下：
注意
Redis3.0最大的改动就是添加Redis的分布式实现Redis Cluster，填补了
Redis官方没有分布式实现的空白。Redis Cluster经历了4年才正式发布也是
有原因的，具体可以参考Redis Cluster的开发日志
（http://antirez.com/news/79）。

1）Redis Cluster：Redis的官方分布式实现。
2）全新的embedded string对象编码结果，优化小对象内存访问，在特定
的工作负载下速度大幅提升。
3）lru算法大幅提升。
4）migrate连接缓存，大幅提升键迁移的速度。
5）migrate命令两个新的参数copy和replace。
6）新的client pause命令，在指定时间内停止处理客户端请求。
7）bitcount命令性能提升。
8）config set设置maxmemory时候可以设置不同的单位（之前只能是字
节），例如config set maxmemory1gb。
9）Redis日志小做调整：日志中会反应当前实例的角色（master或者
slave）。
10）incr命令性能提升。
```

### 4.Redis3.2
Redis3.2在2016年5月6日正式发布，
```
相比于Redis3.0主要特征如下：
1）添加GEO相关功能。
2）SDS在速度和节省空间上都做了优化。
3）支持用upstart或者systemd管理Redis进程。
4）新的List编码类型：quicklist。
5）从节点读取过期数据保证一致性。
6）添加了hstrlen命令。
7）增强了debug命令，支持了更多的参数。
8）Lua脚本功能增强。
9）添加了Lua Debugger。
10）config set支持更多的配置参数。
11）优化了Redis崩溃后的相关报告。
12）新的RDB格式，但是仍然兼容旧的RDB。
13）加速RDB的加载速度。
14）spop命令支持个数参数。
15）cluster nodes命令得到加速。
16）Jemalloc更新到4.0.3版本。
```
### 5.Redis4.0
```
可能出乎很多人的意料，Redis3.2之后的版本是4.0，而不是3.4、3.6、3.8。一般这种重大版本号的升级也意味着软件或者工具本身发生了重大变革，Redis发布了4.0-RC2，下面列出Redis4.0的新特性：
1）提供了模块系统，方便第三方开发者拓展Redis的功能，更多模块详见：http://redismodules.com。
2）PSYNC2.0：优化了之前版本中，主从节点切换必然引起全量复制的问题。
3）提供了新的缓存剔除算法：LFU（Last Frequently Used），并对已有算法进行了优化。
4）提供了非阻塞del和flushall/flushdb功能，有效解决删除bigkey可能造成的Redis阻塞。
5）提供了RDB-AOF混合持久化格式，充分利用了AOF和RDB各自优势。
6）提供memory命令，实现对内存更为全面的监控统计。
7）提供了交互数据库功能，实现Redis内部数据库之间的数据置换。
8）Redis Cluster兼容NAT和Docker。
```
### 6.redis5.0
```
Redis 5主要专注于几个重要功能。相比之下Redis 4非常非常专注于操作类型，
Redis 5的变化大多是面向用户的。
即在现有的基础上增加新的数据类型和操作类型。以下是此版本的主要功能：

Redis 5.0

1.新的流数据类型(Stream data type) https://redis.io/topics/strea...
2.新的 Redis 模块 API：定时器、集群和字典 API(Timers, Cluster and Dictionary APIs)
3.RDB 增加 LFU 和 LRU 信息
4.集群管理器从 Ruby (redis-trib.rb) 移植到了redis-cli 中的 C 语言代码
5.新的有序集合(sorted set)命令：ZPOPMIN/MAX 和阻塞变体(blocking variants)
6.升级 Active defragmentation 至 v2 版本
7.增强 HyperLogLog 的实现
8.更好的内存统计报告
9.许多包含子命令的命令现在都有一个 HELP 子命令
10.客户端频繁连接和断开连接时，性能表现更好
11.许多错误修复和其他方面的改进
12.升级 Jemalloc 至 5.1 版本
13.引入 CLIENT UNBLOCK 和 CLIENT ID
14.新增 LOLWUT 命令 http://antirez.com/news/123
15.在不存在需要保持向后兼容性的地方，弃用 "slave" 术语
16.网络层中的差异优化
17.Lua 相关的改进
18.引入动态的 HZ(Dynamic HZ) 以平衡空闲 CPU 使用率和响应性
19.对 Redis 核心代码进行了重构并在许多方面进行了改进

Redis Stream

Redis stream本质上是个时序数据结构，具有如下特点：

每条记录是结构化、可扩展的对 每条记录在日志中有唯一标识，标识中包含了时间戳信息，单调递增 可以根据需要自动清理历史记录 保存在内存中，支持持久化

底层是修改版的radix tree，每个node存储了一个listpack。listpack是一块连续的内存block，用于序列化msg entry及相关元信息，如msg ID，使用了多种编码，用于节省内存，是ziplist的升级版。如果XADD每次添加的对中的field是一样的，那么field不会重复存储。

Redis Stream使用演示

￼

Redis Stream使用场景

可用作时通信等，大数据分析，异地数据备份等
￼

客户端可以平滑扩展，提高处理能力
￼

Zpop

Sorted Sets 增加了类似List的pop命令：
ZPOPMAX 命令用于移除并弹出有序集合中分值最大的 count 个元素
ZPOPMIN 命令用于移除并弹出有序集合中分值最小的 count 个元素
BZPOPMAX 和 BZPOPMIN 是上述两个命令的阻塞变种.

￼

CLIENT：

Client id返回当前连接的ID，每个ID符合如下约束：

永不重复，可以判断当前链接是否断链过 单调递增，可以判断不同链接的接入顺序

Client unblock：
当客户端因为执行具有阻塞功能的命令（如BRPOP、XREAD或者WAIT）被阻塞时，该命令可以通过其他连接解除客户端的阻塞

￼

Redis 5.0优势：

新增的stream数据结构，丰富的应用场景和想象空间 内核的改进和bugfix，使用更健壮
支持账号体系，根据账号用途赋予相应的权限，更加安全
审计日志，记录了读写操作、敏感操作(keys、flushall等)、慢日志、管理类命令，供用户查询
大key分析，基于快照的完整内存分析，更准确，直接输出内存消耗top排行的key 支持单机和集群版的平滑迁移

```

### 6.redis6.0

```
Redis 6.0 新特性
2020.4.30 Redis作者 antirez 在其 [博客](Redis 6.0.0 GA is out!) 宣布：Redis 6.0.0稳定版本发布了。

简单介绍一下Redis6.0 有哪些重要新特性。
1.多线程IO
Redis 6引入多线程IO，但多线程部分只是用来处理网络数据的读写和协议解析，执行命令仍然是单线程。
之所以这么设计是不想因为多线程而变得复杂，需要去控制 key、lua、事务，LPUSH/LPOP 等等的并发问题。

2.重新设计了客户端缓存功能
实现了Client-side-caching（客户端缓存）功能。放弃了caching slot，而只使用key names。

Redis server-assisted client side caching

3.RESP3协议
RESP（Redis Serialization Protocol）是 Redis 服务端与客户端之间通信的协议。
Redis 5 使用的是 RESP2，而 Redis 6 开始在兼容 RESP2 的基础上，开始支持 RESP3。

推出RESP3的目的：一是因为希望能为客户端提供更多的语义化响应，以开发使用旧协议难以实现的功能；
另一个原因是实现 Client-side-caching（客户端缓存）功能。


4.支持SSL
连接支持SSL，更加安全。

5.ACL权限控制
  1.支持对客户端的权限控制，实现对不同的key授予不同的操作权限。

  2.有一个新的ACL日志命令，允许查看所有违反ACL的客户机、访问不应该访问的命令、
   访问不应该访问的密钥，或者验证尝试失败。这对于调试ACL问题非常有用。

6.提升了RDB日志加载速度
  根据文件的实际组成（较大或较小的值），可以预期20/30%的改进。当有很多客户机连接时，信息也更快了，这是一个老问题，现在终于解决了。

7.发布官方的Redis集群代理模块 Redis Cluster proxy
  在 Redis 集群中，客户端会非常分散，现在为此引入了一个集群代理，
  可以为客户端抽象 Redis 群集，使其像正在与单个实例进行对话一样。同时在简单且客户端仅使用简单命令和功能时执行多路复用。


8.提供了众多的新模块（modules）API

```

详情请参考：

https://www.cnblogs.com/mumage/p/12832766.html