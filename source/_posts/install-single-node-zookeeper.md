---
title: 单节点 zookeeper 安装
date: 2016-08-31 10:15:22
tags:
---

## 安装过程
```
# 下载
cd /opt
wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.8/zookeeper-3.4.8.tar.gz
tar -axvf  zookeeper-3.4.8.tar.gz
cd zookeeper-3.4.8

# 环境变量
cat >> ~/.bashrc <<EOF
##########    ZOOKEEPER  ####################
export ZOOKEEPER_HOME=/opt/zookeeper-3.4.8
export PATH=\$PATH:\$ZOOKEEPER_HOME/bin
export CLASSPATH=\$CLASSPATH:\$ZOOKEEPER_HOME/lib/*
EOF

# 使环境变量生效
source ~/.bashrc
```

```
# 配置
cp $ZOOKEEPER_HOME/conf/zoo_sample.cfg $ZOOKEEPER_HOME/conf/zoo.cfg
# 修改zoo.cfg的datadir配置为
dataDir=/data/var/lib/zookeeper
```

## 启动
```
# 启动
$ZOOKEEPER_HOME/bin/zkServer.sh start
# 关闭
$ZOOKEEPER_HOME/bin/zkServer.sh stop
```



## 参考文档
* <https://zookeeper.apache.org/doc/r3.4.8/>
* <https://zookeeper.apache.org/doc/r3.4.8/zookeeperStarted.html>