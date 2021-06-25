## redisDb结构
/* Redis database representation. There are multiple databases identified
* by integers from 0 (the default database) up to the max configured
* database. The database number is the 'id' field in the structure. */
typedef struct redisDb {
  dict *dict;                 /* The keyspace for this DB */
  dict *expires;              /* Timeout of keys with a timeout set */
  dict *blocking_keys;        /* Keys with clients waiting for data (BLPOP)*/
  dict *ready_keys;           /* Blocked keys that received a PUSH */
  dict *watched_keys;         /* WATCHED keys for MULTI/EXEC CAS */
  int id;                     /* Database ID */
  long long avg_ttl;          /* Average TTL, just for stats */
  list *defrag_later;         /* List of key names to attempt to defrag one by one, gradually. */ 
} redisDb;
  
dict.expires 只存储过期key

## set (key value ex) 调用流程
redisDb.expires存储过期key
```
setExpire db.c:1082
setGenericCommand t_string.c:88
setCommand t_string.c:139
call server.c:2478
processCommand server.c:2785
processInputBuffer networking.c:1466
processInputBufferAndReplicate networking.c:1501
readQueryFromClient networking.c:1583
aeProcessEvents ae.c:443
aeMain ae.c:501
main server.c:4437
start 0x00007fff666bc3d5
```

##  setex (testexkey 1000 111) 调用流程
```
setExpire db.c:1082
setGenericCommand t_string.c:88
setexCommand t_string.c:149
call server.c:2478
processCommand server.c:2785
processInputBuffer networking.c:1466
processInputBufferAndReplicate networking.c:1501
readQueryFromClient networking.c:1583
aeProcessEvents ae.c:443
aeMain ae.c:501
main server.c:4437
start 0x00007fff666bc3d5
```
## expire (testexkey 1000) 调用流程
```
setExpire db.c:1080
expireGenericCommand expire.c:447
expireCommand expire.c:458
call server.c:2478
processCommand server.c:2785
processInputBuffer networking.c:1466
processInputBufferAndReplicate networking.c:1501
readQueryFromClient networking.c:1583
aeProcessEvents ae.c:443
aeMain ae.c:501
main server.c:4437
start 0x00007fff666bc3d5
```