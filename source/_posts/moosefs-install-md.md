---
title: ubuntu下安装MooseFS单机版
date: 2018-04-04 12:26:07
tags:
---

## 介绍
MooseFS，是千兆级的分布式的文件系统，遵循 POSIX 协议。

## 资源
### 官方文档
<https://moosefs.com/>

### github 地址
<https://github.com/moosefs/moosefs>

## 安装步骤

### 添加源

* Add the key:
```
wget -O - https://ppa.moosefs.com/moosefs.key | apt-key add -
```

* Add an appropriate repository entry in `/etc/apt/sources.list.d/moosefs.list`:
```
echo "deb http://ppa.moosefs.com/moosefs-3/apt/ubuntu/xenial xenial main" > /etc/apt/sources.list.d/moosefs.list
```

* Then run:

```
apt update
```

### 安装 moosefs-master

* 安装
```
apt install moosefs-master
```

* 启动
```
mfsmaster start
```

* 添加host: `/etc/hosts`
```
192.168.8.30 mfsmaster
```

### 安装 moosefs-chunkserver

* 安装

```
apt install moosefs-chunkserver
```

* 在`/etc/mfs/mfshdd.cfg` 添加1个或多个路径，作为分区存储数据，例如
```
/data/database/mfs/chunks1
/data/database/mfs/chunks2
/data/database/mfs/chunks3
```

> 建议使用XFS作为底层文件系统

* 创建目录和授权
```
mkdir -p /data/database/mfs/chunks1 /data/database/mfs/chunks2 /data/database/mfs/chunks3
chown mfs:mfs /data/database/mfs/chunks1 /data/database/mfs/chunks2 /data/database/mfs/chunks3
chmod 770 /data/database/mfs/chunks1 /data/database/mfs/chunks2 /data/database/mfs/chunks3
```

* 启动
```
mfschunkserver start
```

### 客户端：挂载MooseFS 文件系统

* 安装
```
apt install moosefs-client
```

* 挂载
```
mkdir /data/mfs
mount -t moosefs mfsmaster: /data/mfs
```

* 可以在`/etc/fstab`下添加MooseFS 的挂载
```
mfsmaster:    /data/mfs    moosefs    defaults,mfsdelayedinit    0 0
```

* 现在，可以切换到 `/data/mfs` 管理你的文件

### web 监控（可选）

* 安装
```
apt install moosefs-cgi moosefs-cgiserv moosefs-cli
```

* 启动web监控
```
mfscgiserv start
```
在<http://mfsmaster:9425> 查看moose文件系统情况

### moosefs-metalogger（可选，建议）

* 强烈建议在不是master节点的机器上安装至少一个Metalogger, Metalogger 不断地同步和备份元数据。

* 安装
```
apt install moosefs-metalogger
``

* 启动
```
mfsmetalogger start
```