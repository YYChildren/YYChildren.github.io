---
title: impala 解析json
date: 2017-10-17 22:42:20
tags:
---

## 前提
直接通过hql查询字段为json字符串里的值，比如
```
获取'{"keyword":1}' 的keyword值 1
```

## 网上相关的文章
* **相关issue**： <https://issues.apache.org/jira/browse/IMPALA-376>

* 这篇issue提到impala官方仍然为实现impala的json函数，但是评论中有提到2中方法
1. 直接引用hive的udf
2. 开源的实现：  <https://github.com/nazgul33/impala-get-json-object-udf>

* 为了简单起见，我直接使用了第一种方案，impala引入HiveUDF的方法参考： <https://www.cloudera.com/documentation/enterprise/latest/topics/impala_udf.html#udfs_hive>

* 该函数的使用方法参考：<https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF> 

## 在CDH 集群上的实现 
```
sudo -u impala hadoop fs -mkdir -p /user/impala/udf
sudo -u impala hadoop fs -put -f /opt/cloudera/parcels/CDH/jars/hive-exec-*.jar /user/impala/udf/hive-exec.jar
impala-shell -q "
CREATE DATABASE IF NOT EXISTS udf;
USE udf;
DROP FUNCTION IF EXISTS udf.get_json_object(string, string);
CREATE FUNCTION udf.get_json_object(string, string) returns string location '/user/impala/udf/hive-exec.jar' symbol='org.apache.hadoop.hive.ql.udf.UDFJson';
"
```

## 测试
```
Query: select udf.get_json_object('{"keyword":1}', '$.keyword')
+---------------------------------------------------+
| udf.get_json_object('{"keyword":1}', '$.keyword') |
+---------------------------------------------------+
| 1                                                 |
+---------------------------------------------------+
```

