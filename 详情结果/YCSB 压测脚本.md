# 监测资源占用情况

```
date "+%H:%M:%S">top_monitor.txt|top -u scylla -d 1 -b | grep --line-buffered "83248 scylla" >> top_monitor.txt &
```

```
# 退出 shell 不停止脚本, -h 表示不从 jobs 中移除
disown -h {PID}
```

# REDIS

```
python bin/ycsb load redis -s -P workloads/workloada -p "redis.host=192.168.1.23" -p "redis.port=6379" -p "redis.password=your_password" -threads 100 > redis_load.txt 2>&1 &
```

```
python bin/ycsb run redis -s -P workloads/workloada -p "redis.host=192.168.1.23" -p "redis.port=6379" -p "redis.password=your_password" -threads 100 > redis_runA.txt 2>&1 &
```

# MONGODB

```
python bin/ycsb load mongodb -s -P workloads/workloada -p mongodb.url=mongodb://your_username:your_password@192.168.1.23:27017/ycsb?w=0 -threads 100 > mongodb_load.txt 2>&1 &
```

```
python bin/ycsb run mongodb -s -P workloads/workloada -p mongodb.url=mongodb://your_username:your_password@192.168.1.23:27017/ycsb?w=0 -threads 100 > mongodb_runA.txt 2>&1 &
```

# SCYLLA

```
python bin/ycsb load cassandra-cql -s -P workloads/workloada -p "hosts=192.168.1.23" -p "port=9042" -p "cassandra.keyspace=ycsb" -p "cassandra.username=your_username" -p "cassandra.password=your_password" -threads 100  > scylla_load.txt 2>&1 &
```

```
python bin/ycsb run cassandra-cql -s -P workloads/workloada -p "hosts=192.168.1.23" -p "port=9042" -p "cassandra.keyspace=ycsb" -p "cassandra.username=cassandra" -p "cassandra.password=your_password" -threads 100 > scylla_runA.txt 2>&1 &
```