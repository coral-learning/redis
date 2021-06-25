## SET

SET key value [EX seconds] [PX milliseconds] [NX|XX]

将字符串值 value 关联到 key 。

如果 key 已经持有其他值， SET 就覆写旧值，无视类型。

对于某个原本带有生存时间（TTL）的键来说，

当 SET 命令成功在这个键上执行时， 这个键原有的 TTL 将被清除。

### 可选参数

```
从 Redis 2.6.12 版本开始， SET 命令的行为可以通过一系列参数来修改：
EX second ：设置键的过期时间为 second 秒。 SET key value EX second 效果等同于 SETEX key second value 。
PX millisecond ：设置键的过期时间为 millisecond 毫秒。 SET key value PX millisecond 效果等同于 PSETEX key millisecond value 。
NX ：只在键不存在时，才对键进行设置操作。 SET key value NX 效果等同于 SETNX key value 。
XX ：只在键已经存在时，才对键进行设置操作。
因为 SET 命令可以通过参数来实现和 SETNX 、 SETEX 和 PSETEX 三个命令的效果，
所以将来的 Redis 版本可能会废弃并最终移除 SETNX 、 SETEX 和 PSETEX 这三个命令。

可用版本：
>= 1.0.0
时间复杂度：
O(1)
返回值：
在 Redis 2.6.12 版本以前， SET 命令总是返回 OK 。
```

```
从 Redis 2.6.12 版本开始， SET 在设置操作成功完成时，才返回 OK 。
如果设置了 NX 或者 XX ，但因为条件没达到而造成设置操作未执行，那么命令返回空批量回复（NULL Bulk Reply）。
```

### 对不存在的键进行设置

```
redis 127.0.0.1:6379> SET testkey "value"
OK

redis 127.0.0.1:6379> GET testkey
"value"
```

### 对已存在的键进行设置

```
redis 127.0.0.1:6379> SET testkey "new-value"
OK

redis 127.0.0.1:6379> GET testkey
"new-value"
```

### 使用 EX 选项

设置键的过期时间为 second 秒

```
redis 127.0.0.1:6379> SET testexkey "hello" EX 10000
OK

redis 127.0.0.1:6379> GET testexkey
"hello"

redis 127.0.0.1:6379> TTL testexkey
(integer) 9986
```

### 使用 PX 选项

设置键的过期时间为 millisecond 毫秒

```
redis 127.0.0.1:6379> SET testpxkey "moto" PX 100000
OK

redis 127.0.0.1:6379> GET testpxkey
"moto"

redis 127.0.0.1:6379> PTTL testpxkey
(integer) 83818

```

### 使用 NX 选项

只在键不存在时，才对键进行设置操作

```
redis 127.0.0.1:6379> SET testnxkey "value" NX
OK      # 键不存在，设置成功

redis 127.0.0.1:6379> GET testnxkey
"value"

redis 127.0.0.1:6379> SET testnxkey "new-value" NX
(nil)   # 键已经存在，设置失败

redis 127.0.0.1:6379> GET testnxkey
"value" # 维持原值不变
```

### 使用 XX 选项

只在键已经存在时，才对键进行设置操作

```
redis 127.0.0.1:6379> EXISTS testxxkey
(integer) 0

redis 127.0.0.1:6379> SET testxxkey "value" XX
(nil)   # 因为键不存在，设置失败

redis 127.0.0.1:6379> SET testxxkey "value"
OK      # 先给键设置一个值

redis 127.0.0.1:6379> SET testxxkey "new-value" XX
OK      # 设置新值成功

redis 127.0.0.1:6379> GET testxxkey
"new-value"
```

### NX 或 XX 可以和 EX 或者 PX 组合使用

```
redis 127.0.0.1:6379> SET testexxxkey "hello" EX 10086 NX
OK

redis 127.0.0.1:6379> GET testexxxkey
"hello"

redis 127.0.0.1:6379> TTL testexxxkey
(integer) 10063

redis 127.0.0.1:6379> SET testexxxkey "old value"
OK

redis 127.0.0.1:6379> SET testexxxkey "new value" PX 123321
OK

redis 127.0.0.1:6379> GET testexxxkey
"new value"

redis 127.0.0.1:6379> PTTL testexxxkey
(integer) 112999
```

### EX 和 PX 可以同时出现，但后面给出的选项会覆盖前面给出的选项

```
redis 127.0.0.1:6379> SET key "value" EX 1000 PX 5000000
OK

redis 127.0.0.1:6379> TTL key
(integer) 4993  # 这是 PX 参数设置的值

redis 127.0.0.1:6379> SET another-key "value" PX 5000000 EX 1000
OK

redis 127.0.0.1:6379> TTL another-key
(integer) 997   # 这是 EX 参数设置的值
使用模式
命令 SET resource-name anystring NX EX max-lock-time 是一种在 Redis 中实现锁的简单方法。
```

客户端执行以上的命令：

```
如果服务器返回 OK ，那么这个客户端获得锁。
如果服务器返回 NIL ，那么客户端获取锁失败，可以在稍后再重试。
设置的过期时间到达之后，锁将自动释放。
可以通过以下修改，让这个锁实现更健壮：
不使用固定的字符串作为键的值，而是设置一个不可猜测（non-guessable）的长随机字符串，作为口令串（token）。
不使用 DEL 命令来释放锁，而是发送一个 Lua 脚本，这个脚本只在客户端传入的值和键的口令串相匹配时，才对键进行删除。
这两个改动可以防止持有过期锁的客户端误删现有锁的情况出现。
```

以下是一个简单的解锁脚本示例：

```
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```
这个脚本可以通过 EVAL ...script... 1 resource-name token-value 命令来调用。

### 源码分析
* 流程调用以redis5.0版本为例，set command流程（倒序调用debug偷懒）如下：

```
start 0x00007fff6957ecc9
main server.c:4437
    //传入server调用事件
aeMain ae.c:501
    //处理网络事件
aeProcessEvents ae.c:443
    //数据转发
readQueryFromClient networking.c:1583
    //  
processInputBufferAndReplicate networking.c:1501
    //任务分发
processInputBuffer networking.c:1466
    //调用processCommand
processCommand server.c:2785
    //处理特殊命令，命令分类及执行方式，如是否要加入队列
    //getNodeByQuery(获取节点与当前节点对比,判断该节点是否存储该数据，存在执行正常流程，不存在通过 clusterRedirectClient函数返回客户端
    //包含key不存在，节点挂了，已经迁移等情况
call server.c:2478
    //进行计时调用，c->cmd->proc(c);分发执行调用setCommand，进行慢查询日志记录，只记录大于配置时间的
setCommand t_string.c:139
    //调用setGenericCommand【数据类型】
setGenericCommand t_string.c:86
    //调用setKey【数据类型】
setKey db.c:218
    //决定为dbadd or dboverwrite，计数器自增，删除过期key，key通知
dbAdd db.c:175
    //调用dictAdd，处理集群slotToKeyAdd
dictAdd dict.c:267
    //代码较为简单，调用dictAddRaw，设置val
dictAddRaw dict.c:317
    //处理渐进hash,获取索引,选用ht,存储ht->table
    

```
