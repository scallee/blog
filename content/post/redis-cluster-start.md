---
title: "Redis Cluster Start"
date: 2019-06-16T20:58:58+08:00
lastmod: 2019-06-16T20:58:58+08:00
draft: false
keywords: ["redis","linux"]
description: "最简单的redis集群cluster模式启动脚本"
tags: ["redis","linux"]
categories: ["redis"]
# author: ""
comment: false
toc: true
contentCopyright: false
reward: false
mathjax: false
---

Redis集群Cluster模式启动脚本
<!--more-->

```sh
#!/bin/bash
redis_path="/usr/local/cluster/"
cluster_num=`ps aux | grep redis |  wc -l`
if [ "${cluster_num}" -le 0 ]
then
        echo -e "===== Success: Has killed all cluster progress."
else
		pkill -9 redis
fi

cd ${redis_path}
rm -rf ${redis_path}700*/appendonly.aof
rm -rf ${redis_path}700*/dump.rdb
rm -rf ${redis_path}700*/nodes*
rm -rf ${redis_path}nodes*
rm -rf ${redis_path}appendonly.aof
cd /usr/local/cluster/7000/
redis-server /usr/local/cluster/7000/redis.conf
cd /usr/local/cluster/7001/
redis-server /usr/local/cluster/7001/redis.conf
cd /usr/local/cluster/7002/
redis-server /usr/local/cluster/7002/redis.conf
cd /usr/local/cluster/7003/
redis-server /usr/local/cluster/7003/redis.conf
cd /usr/local/cluster/7004/
redis-server /usr/local/cluster/7004/redis.conf
cd /usr/local/cluster/7005/
redis-server /usr/local/cluster/7005/redis.conf

cd /usr/local/redis5.0/redis-5.0.0/src/
echo yes | ./redis-cli  --cluster  create   192.168.110.128:7000 192.168.110.128:7001 192.168.110.128:7002 192.168.110.128:7003 192.168.110.128:7004 192.168.110.128:7005 --cluster-replicas 1 

#试试echo yes > 
```