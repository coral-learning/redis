## redis

### build
```
tar 解压
cd deps
make lua hiredis linenoise

cd ..
make 
make install

```

### start 

vim restart.sh
```
ps -ef | grep redis-server | grep -v grep | awk '{print $2}' | xargs kill -9
./src/redis-server redis.conf --daemonize yes

```