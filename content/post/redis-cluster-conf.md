---
title: "Redis Cluster Conf"
date: 2019-06-17T10:03:42+08:00
lastmod: 2019-06-17T10:03:42+08:00
draft: false
keywords: ["redis","linux"]
description: "redis-cluster设置"
tags: ["redis","linux"]
categories: ["redis"]
# author: ""
comment: false
toc: true
contentCopyright: false
reward: false
mathjax: false
---

redis集群简单设置脚本
<!--more-->

```sh

#!/bin/bash

# redis-cluster配置
CULSTER_CONF=/usr/local/cluster/redis.conf

CLUSTER_BASE_PATH=/usr/local/cluster/700

for i in $(seq 0 5)
do
mkdir -p $CLUSTER_BASE_PATH$i;
done
echo 复制redis.conf 到目标目录
cp /usr/local/redis-5.0.0/redis.conf /usr/local/cluster/
echo redis.conf 复制完成
echo 修改redis.conf参数
sed -i 's/port 6379/port 7000/g' $CULSTER_CONF
sed -i 's/# cluster-enabled yes/cluster-enabled yes/g' $CULSTER_CONF
sed -i 's/# cluster-config-file nodes-6379.conf/cluster-config-file nodes-7000.conf/g' $CULSTER_CONF
sed -i 's/# cluster-node-timeout 15000/cluster-node-timeout 15000/g' $CULSTER_CONF
sed -i 's/appendonly no/appendonly yes/g' $CULSTER_CONF
sed -i 's/dbfilename dump.rdb/dbfilename dump-7000.rdb/g' $CULSTER_CONF

echo 完成修改redis.conf

# Linux批量复制文件到文件夹
echo 将修改后的redis.conf循环复制到cluster下的目录里
echo ${CLUSTER_BASE_PATH}* | xargs -n 1 cp -v $CULSTER_CONF

echo 针对每个cluster修改相应的redis.conf参数
for i in $(seq 1 5)
do
sed -i  "s/port 7000/port 700$i/g"  $CLUSTER_BASE_PATH$i/redis.conf
sed -i  "s/cluster-config-file nodes-7000.conf/cluster-config-file nodes-700$i.conf/g"  $CLUSTER_BASE_PATH$i/redis.conf
sed -i  "s/dbfilename dump-7000.rdb/dbfilename dump-700$i.rdb/g"   $CLUSTER_BASE_PATH$i/redis.conf
done
echo 参数修改完成
```