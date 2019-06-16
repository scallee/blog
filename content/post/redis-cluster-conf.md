---
title: "Redis Cluster Conf"
date: 2019-06-17T10:03:42+08:00
lastmod: 2019-06-17T10:03:42+08:00
draft: false
keywords: ["redis","linux"]
description: "redis-cluster设置"
tags: ["redis","linux","cluster"]
categories: ["redis"]
# author: ""
comment: false
toc: true
contentCopyright: false
reward: false
mathjax: false
---

redis集群简单设置脚本

未完成
<!--more-->

```sh
#!/bin/bash


CULSTER_CONF=/usr/local/cluster/redis.conf

CLUSTER_BASE_PATH=/usr/local/cluster/700

for i in $(seq 0 5)
do
mkdir -p $CLUSTER_BASE_PATH$i;
done


cp /usr/local/redis-5.0.0/redis.conf /usr/local/cluster/

sleep 1 
echo 复制redis.conf
sed -i 's/port 6379/port 7000/g' $CULSTER_CONF
sed -i 's/# cluster-enabled yes/cluster-enabled yes/g' $CULSTER_CONF
sed -i 's/# cluster-config-file nodes-6379.conf/cluster-config-file nodes-7000.conf/g' $CULSTER_CONF
sed -i 's/# cluster-iode-timeout 15000/cluster-iode-timeout 15000/g' $CULSTER_CONF
sed -i 's/appendonly no/appendonly yes/g' $CULSTER_CONF
sed -i 's/dbfilename dump.rdb/dbfilename dump-7000.rdb/g' $CULSTER_CONF
sleep 1 
echo 修改redis.conf

# Linux批量复制文件到文件夹

echo ${CLUSTER_BASE_PATH}* | xargs -n 1 cp -v $CULSTER_CONF

for i in $(seq 1 5)
do
sed -i 's/port 7000/port 7001/g' $CLUSTER_BASE_PATH$i/redis.conf
sed -i 's/cluster-config-file nodes-7000.conf/cluster-config-file nodes-7001.conf/g' $CLUSTER_BASE_PATH$i/redis.conf
sed -i 's/dbfilename dump-7000.rdb/dbfilename dump-7001.rdb/g'  $CLUSTER_BASE_PATH$i/redis.conf
done

```