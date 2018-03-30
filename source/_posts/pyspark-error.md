---
title: pyspark 报错问题
date: 2017-04-01 11:29:49
tags:
---

## 运行代码

```
# pyspark
>>> textFile = sc.textFile("/tmp/1.sh")
>>> textFile.count()
```

## 报错

```
Python in worker has different version 2.6 than that in driver 2.7, PySpark cannot run with different minor versions
```

## 原因
* pytspark 启动时使用的是python2.7
* 内部脚本设置的PYSPARK_PYTHON 是python2.6

## 解决
增加 config/spark-env.sh 配置
```
PYSPARK_PYTHON=/usr/local/bin/python
```

## CDH 解决办法
![](https://leanote.com/api/file/getImage?fileId=58df1f8cab64413a33001d46)