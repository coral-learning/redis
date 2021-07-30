## 1.cluster命令
命令                           | 描述            | 示例                            |  备注 
------------                  |--------------   |----------                      |  -----
CLUSTER HELP                  | 支持命令及描述     |  CLUSTER HELP                  |   
CLUSTER ADDSLOTS              | 为当前节点分配槽位  | CLUSTER ADDSLOTS 0 5          | 
CLUSTER BUMPEPOCH             | 推进集群配置纪元    | CLUSTER BUMPEPOCH             |
CLUSTER COUNT-failure-reports | 返回的失败报告数量  | CLUSTER COUNT-failure-reports node-id |
CLUSTER COUNTKEYSINSLOT       | 返回<槽位>中的键数  |  CLUSTER COUNTKEYSINSLOT 1    |  
CLUSTER DELSLOTS              | 删除当前节点的槽位信息| CLUSTER DELSLOTS 2 5        | 
CLUSTER FAILOVER              | 集群故障转移       | CLUSTER FAILOVER             |
CLUSTER FORGET                | 删除节点          | CLUSTER FORGET node-id      |
CLUSTER GETKEYSINSLOT         | 返回当前节点存储在slot中的键名 |  CLUSTER GETKEYSINSLOT 1000 3 | 
CLUSTER FLUSHSLOTS            | 删除当前节点自己的槽位信息     | CLUSTER FLUSHSLOTS |
CLUSTER INFO                  | 返回集群信息                | CLUSTER INFO    |
CLUSTER KEYSLOT               | 返回的哈希槽                | CLUSTER KEYSLOT 1 |
CLUSTER MEET                  | 将节点连接到一个工作集群       | CLUSTER MEET 10.3.4.111 7001 |
CLUSTER MYID                  | 返回当前节点ID               | CLUSTER MYID   |
CLUSTER NODES                 | 返回集群节点信息              | CLUSTER NODES  |
CLUSTER REPLICATE             | 将当前节点配置为副本              | CLUSTER REPLICATE node-id |
CLUSTER RESET                 | 重置当前节点              | CLUSTER RESET   |
CLUSTER SETSLOT               | 修改接受节点中哈希槽的状态  | CLUSTER SETSLOT 1  MIGRATING node-id|
CLUSTER REPLICAS              | 返回节点REPLICAS         | CLUSTER REPLICAS node-id |
CLUSTER SET-CONFIG-EPOCH      | 修改接受节点中哈希槽的状态  | CLUSTER SET-CONFIG-EPOCH 1|
CLUSTER SLOTS                 | 返回节点槽位                | CLUSTER SLOTS     | 


```
 1) CLUSTER <subcommand> arg arg ... arg. Subcommands are:
 2) ADDSLOTS <slot> [slot ...] -- Assign slots to current node.
 3) BUMPEPOCH -- Advance the cluster config epoch.
 4) COUNT-failure-reports <node-id> -- Return number of failure reports for <node-id>.
 5) COUNTKEYSINSLOT <slot> - Return the number of keys in <slot>.
 6) DELSLOTS <slot> [slot ...] -- Delete slots information from current node.
 7) FAILOVER [force|takeover] -- Promote current replica node to being a master.
 8) FORGET <node-id> -- Remove a node from the cluster.
 9) GETKEYSINSLOT <slot> <count> -- Return key names stored by current node in a slot.
10) FLUSHSLOTS -- Delete current node own slots information.
11) INFO - Return onformation about the cluster.
12) KEYSLOT <key> -- Return the hash slot for <key>.
13) MEET <ip> <port> [bus-port] -- Connect nodes into a working cluster.
14) MYID -- Return the node id.
15) NODES -- Return cluster configuration seen by node. Output format:
16)     <id> <ip:port> <flags> <master> <pings> <pongs> <epoch> <link> <slot> ... <slot>
17) REPLICATE <node-id> -- Configure current node as replica to <node-id>.
18) RESET [hard|soft] -- Reset current node (default: soft).
19) SET-config-epoch <epoch> - Set config epoch of current node.
20) SETSLOT <slot> (importing|migrating|stable|node <node-id>) -- Set slot state.
21) REPLICAS <node-id> -- Return <node-id> replicas.
22) SLOTS -- Return information about slots range mappings. Each range is made of:
23)     start, end, master and replicas IP addresses, ports and ids
```

* 详细参考：http://doc.redisfans.com/

## 2.cluster命令解析

### 2.1.cluster info

```
clusterGenNodesDescription cluster.c:4122
clusterCommand cluster.c:4305
call server.c:2478
processCommand server.c:2785
processInputBuffer networking.c:1466
processInputBufferAndReplicate networking.c:1501
readQueryFromClient networking.c:1583
aeProcessEvents ae.c:443
aeMain ae.c:501
main server.c:4437
start 0x00007fff63f8e3d5

```

## 附录:主要函数列表

```
clusterNode *createClusterNode(char *nodename, int flags);
int clusterAddNode(clusterNode *node);
void clusterAcceptHandler(aeEventLoop *el, int fd, void *privdata, int mask);
void clusterReadHandler(aeEventLoop *el, int fd, void *privdata, int mask);
void clusterSendPing(clusterLink *link, int type);
void clusterSendFail(char *nodename);
void clusterSendFailoverAuthIfNeeded(clusterNode *node, clusterMsg *request);
void clusterUpdateState(void);
int clusterNodeGetSlotBit(clusterNode *n, int slot);
sds clusterGenNodesDescription(int filter);
clusterNode *clusterLookupNode(const char *name);
int clusterNodeAddSlave(clusterNode *master, clusterNode *slave);
int clusterAddSlot(clusterNode *n, int slot);
int clusterDelSlot(int slot);
int clusterDelNodeSlots(clusterNode *node);
int clusterNodeSetSlotBit(clusterNode *n, int slot);
void clusterSetMaster(clusterNode *n);
void clusterHandleSlaveFailover(void);
void clusterHandleSlaveMigration(int max_slaves);
int bitmapTestBit(unsigned char *bitmap, int pos);
void clusterDoBeforeSleep(int flags);
void clusterSendUpdate(clusterLink *link, clusterNode *node);
void resetManualFailover(void);
void clusterCloseAllSlots(void);
void clusterSetNodeAsMaster(clusterNode *n);
void clusterDelNode(clusterNode *delnode);
sds representClusterNodeFlags(sds ci, uint16_t flags);
uint64_t clusterGetMaxEpoch(void);
int clusterBumpConfigEpochWithoutConsensus(void);
void moduleCallClusterReceivers(const char *sender_id, uint64_t module_id, uint8_t type, const unsigned char *payload, uint32_t len);
```