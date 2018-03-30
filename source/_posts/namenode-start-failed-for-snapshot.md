---
title: Hadoop NameNode 无法启动：删除SnapShot所导致
date: 2017-02-05 20:25:37
tags:
---

## 事件回顾
2017-02-03 14:14 时看到Hadoop Namenode的swap占用较高，于是尝试重启NameNode以释放Swap。想不到竟然无法启动。

日志如下：

```
2017-02-03 14:15:09,218 ERROR org.apache.hadoop.hdfs.server.namenode.NameNode: Failed to start namenode.
java.lang.NullPointerException
    at org.apache.hadoop.hdfs.server.namenode.INodeDirectory.addChild(INodeDirectory.java:531)
    at org.apache.hadoop.hdfs.server.namenode.FSImageFormatPBINode$Loader.addToParent(FSImageFormatPBINode.java:252)
    at org.apache.hadoop.hdfs.server.namenode.FSImageFormatPBINode$Loader.loadINodeDirectorySection(FSImageFormatPBINode.java:202)
    at org.apache.hadoop.hdfs.server.namenode.FSImageFormatProtobuf$Loader.loadInternal(FSImageFormatProtobuf.java:261)
    at org.apache.hadoop.hdfs.server.namenode.FSImageFormatProtobuf$Loader.load(FSImageFormatProtobuf.java:180)
    at org.apache.hadoop.hdfs.server.namenode.FSImageFormat$LoaderDelegator.load(FSImageFormat.java:226)
    at org.apache.hadoop.hdfs.server.namenode.FSImage.loadFSImage(FSImage.java:929)
    at org.apache.hadoop.hdfs.server.namenode.FSImage.loadFSImage(FSImage.java:913)
    at org.apache.hadoop.hdfs.server.namenode.FSImage.loadFSImageFile(FSImage.java:732)
    at org.apache.hadoop.hdfs.server.namenode.FSImage.loadFSImage(FSImage.java:668)
    at org.apache.hadoop.hdfs.server.namenode.FSImage.recoverTransitionRead(FSImage.java:281)
    at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.loadFSImage(FSNamesystem.java:1061)
    at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.loadFromDisk(FSNamesystem.java:765)
    at org.apache.hadoop.hdfs.server.namenode.NameNode.loadNamesystem(NameNode.java:604)
    at org.apache.hadoop.hdfs.server.namenode.NameNode.initialize(NameNode.java:663)
    at org.apache.hadoop.hdfs.server.namenode.NameNode.<init>(NameNode.java:830)
    at org.apache.hadoop.hdfs.server.namenode.NameNode.<init>(NameNode.java:814)
    at org.apache.hadoop.hdfs.server.namenode.NameNode.createNameNode(NameNode.java:1507)
    at org.apache.hadoop.hdfs.server.namenode.NameNode.main(NameNode.java:1575)
2017-02-03 14:15:09,238 INFO org.apache.hadoop.util.ExitUtil: Exiting with status 1
```

## Hadoop 版本信息
我们当时使用的Hadoop版本为CDH5.4.9

## 问题发现
1. Google之，得到Apache 官方有提到该Bug
    地址：<https://issues.apache.org/jira/browse/HDFS-9406>
2. 另外CDH官方有提到解决该问题的版本
    地址：<https://www.cloudera.com/documentation/enterprise/release-notes/topics/cdh_rn_fixed_in_54.html#fixed_issues_5410>
3. 问题解释
    当删除一个包含INode的最新记录的快照时，fsimage有可能崩溃，因为更早快照的diff的create-list 和 父亲INodeDirectory的child list没有清理。

    >When deleting a snapshot that contains the last record of a given INode, the fsimage may become corrupt because the create list of the snapshot diff in the previous snapshot and the child list of the parent INodeDirectory are not cleaned.

## 解决过程
1. 尝试以 2017-02-03 12 时的 fsimage 恢复，发现问题依旧。
2. 升级CDH 版本到5.4.11，发现问题依旧
3. 继续升级版本到5.7.5，发现问题依旧
4. 查看修复版本的关键文件 INodeDirectory.java、FSImageFormatPBINode.java、FSImageFormatProtobuf.java（位于[hadoop-common项目](https://github.com/cloudera/hadoop-common)），发现并没有对空指针异常做处理的代码。
5. 查看[patch](https://issues.apache.org/jira/secure/attachment/12785614/HDFS-9406.branch-2.7.patch)，实际上也只是让删除快照变得正常，但是对已经出问题的fsimage没有做好兼容。
![patch关键位置](https://leanote.com/api/file/getImage?fileId=58971ff7ab64413f920015d1)
6. 针对[CDH5.7.5-release](https://github.com/cloudera/hadoop-common/tree/cdh5.7.5-release)，尝试修改代码，在Namenode启动时忽略Null 的INode，主要改动如下
    ![](https://leanote.com/api/file/getImage?fileId=58972421ab64413b8000166c)
    编译替换后发现了新的异常：
    ```
    2017-02-04 15:41:08,581 ERROR org.apache.hadoop.hdfs.server.namenode.FSImage: Failed to load image from FSImageFile(file=/data/dfs/nn/current/fsimage_0000000005899107393, cpktTxId=0000000005899107393)
    java.io.IOException: Cannot find an INode associated with the INode FlumeData.1482923297645 in created list while loading FSImage.
            at org.apache.hadoop.hdfs.server.namenode.snapshot.SnapshotFSImageFormat.loadCreated(SnapshotFSImageFormat.java:158)
            at org.apache.hadoop.hdfs.server.namenode.snapshot.FSImageFormatPBSnapshot$Loader.loadCreatedList(FSImageFormatPBSnapshot.java:244)
            at org.apache.hadoop.hdfs.server.namenode.snapshot.FSImageFormatPBSnapshot$Loader.loadDirectoryDiffList(FSImageFormatPBSnapshot.java:338)
            at org.apache.hadoop.hdfs.server.namenode.snapshot.FSImageFormatPBSnapshot$Loader.loadSnapshotDiffSection(FSImageFormatPBSnapshot.java:189)
            at org.apache.hadoop.hdfs.server.namenode.FSImageFormatProtobuf$Loader.loadInternal(FSImageFormatProtobuf.java:271)
            at org.apache.hadoop.hdfs.server.namenode.FSImageFormatProtobuf$Loader.load(FSImageFormatProtobuf.java:181)
            at org.apache.hadoop.hdfs.server.namenode.FSImageFormat$LoaderDelegator.load(FSImageFormat.java:226)
            at org.apache.hadoop.hdfs.server.namenode.FSImage.loadFSImage(FSImage.java:948)
            at org.apache.hadoop.hdfs.server.namenode.FSImage.loadFSImage(FSImage.java:932)
            at org.apache.hadoop.hdfs.server.namenode.FSImage.loadFSImageFile(FSImage.java:751)
            at org.apache.hadoop.hdfs.server.namenode.FSImage.loadFSImage(FSImage.java:682)
            at org.apache.hadoop.hdfs.server.namenode.FSImage.recoverTransitionRead(FSImage.java:291)
            at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.loadFSImage(FSNamesystem.java:1096)
            at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.loadFromDisk(FSNamesystem.java:778)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.loadNamesystem(NameNode.java:609)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.initialize(NameNode.java:670)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.<init>(NameNode.java:838)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.<init>(NameNode.java:817)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.createNameNode(NameNode.java:1537)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.main(NameNode.java:1605)
    2017-02-04 15:41:08,657 WARN org.apache.hadoop.hdfs.server.namenode.FSNamesystem: Encountered exception loading fsimage
    java.io.IOException: Failed to load FSImage file, see error(s) above for more info.
            at org.apache.hadoop.hdfs.server.namenode.FSImage.loadFSImage(FSImage.java:697)
            at org.apache.hadoop.hdfs.server.namenode.FSImage.recoverTransitionRead(FSImage.java:291)
            at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.loadFSImage(FSNamesystem.java:1096)
            at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.loadFromDisk(FSNamesystem.java:778)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.loadNamesystem(NameNode.java:609)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.initialize(NameNode.java:670)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.<init>(NameNode.java:838)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.<init>(NameNode.java:817)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.createNameNode(NameNode.java:1537)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.main(NameNode.java:1605)
    2017-02-04 15:41:08,660 INFO org.mortbay.log: Stopped HttpServer2$SelectChannelConnectorWithSafeStartup@0.0.0.0:50070
    2017-02-04 15:41:08,761 INFO org.apache.hadoop.metrics2.impl.MetricsSystemImpl: Stopping NameNode metrics system...
    2017-02-04 15:41:08,761 INFO org.apache.hadoop.metrics2.impl.MetricsSystemImpl: NameNode metrics system stopped.
    2017-02-04 15:41:08,761 INFO org.apache.hadoop.metrics2.impl.MetricsSystemImpl: NameNode metrics system shutdown complete.
    2017-02-04 15:41:08,762 ERROR org.apache.hadoop.hdfs.server.namenode.NameNode: Failed to start namenode.
    java.io.IOException: Failed to load FSImage file, see error(s) above for more info.
            at org.apache.hadoop.hdfs.server.namenode.FSImage.loadFSImage(FSImage.java:697)
            at org.apache.hadoop.hdfs.server.namenode.FSImage.recoverTransitionRead(FSImage.java:291)
            at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.loadFSImage(FSNamesystem.java:1096)
            at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.loadFromDisk(FSNamesystem.java:778)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.loadNamesystem(NameNode.java:609)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.initialize(NameNode.java:670)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.<init>(NameNode.java:838)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.<init>(NameNode.java:817)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.createNameNode(NameNode.java:1537)
            at org.apache.hadoop.hdfs.server.namenode.NameNode.main(NameNode.java:1605)
    2017-02-04 15:41:08,764 INFO org.apache.hadoop.util.ExitUtil: Exiting with status 1
    2017-02-04 15:41:08,765 INFO org.apache.hadoop.hdfs.server.namenode.NameNode: SHUTDOWN_MSG:
    ```
7. 根据提示，继续修改代码 
    ![](https://leanote.com/api/file/getImage?fileId=58972421ab64413b8000166b)
    ![](https://leanote.com/api/file/getImage?fileId=58972421ab64413b8000166d)
8. 重新编译修改后的代码，采用的Java版本为<http://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5.7.5/RPMS/x86_64/oracle-j2sdk1.7-1.7.0+update67-1.x86_64.rpm>，编译前需要安装protobuf-2.5.0和protobuf-compile-2.5.0，编译指令为
    ```
    mvn package -Pdist -DskipTests -Dtar
    ```
9. 替换集群NameNode和SecondaryNamenode 下的 /opt/cloudera/parcels/CDH/jars/hadoop-hdfs-2.6.0-cdh5.7.5.jar 为上面编译后的 hadoop-common-cdh5.7.5-release/hadoop-hdfs-project/hadoop-hdfs/target/hadoop-hdfs-2.6.0-cdh5.7.5.jar 
10. 启动NameNode 成功
## 修复总结
1. 目前暂时通过忽略空节点的方式来绕过fsimage的问题，但是由于对hadoop了解还不够深入，不确定会不会引入新的问题，建议最好启动后备份数据，然后重整集群再恢复。
2. 备份一定要做好，开源的方案有可能存在bug，导致各种问题。要做到无法何种故障，都有恢复的手段。

> 最后重点感谢公司强大的大佬和强大运维支持。













