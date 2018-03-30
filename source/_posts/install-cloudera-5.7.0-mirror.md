---
title: 安装Cloudera 5.7.0镜像
date: 2016-05-04 19:34:53
tags:
---

## 镜像包含
* CM5
* CDH5

## 安装Apache HTTP server
```
yum install httpd
# 查看版本
httpd -v
# Server version: Apache/2.2.15 (Unix)
# Server built:   Mar 22 2016 19:03:53
```

## ssl相关模块
```
yum install -y openssl mod_ssl 
```

## 启动http服务器
**注意：**这里暂时忽略证书
```
service httpd start
```

## CM与CDH
**CM 地址**

[cm5](https://archive.cloudera.com/cm5/repo-as-tarball)  [5.7.0](http://archive.cloudera.com/cm5/repo-as-tarball/5.7.0/)

**CDH 地址**
[CDH5 rpm形式](http://archive.cloudera.com/cm5/repo-as-tarball/)  [5.7.0](http://archive.cloudera.com/cdh5/repo-as-tarball/5.7.0/)
[CDH5 parcel](http://archive.cloudera.com/cdh5/parcels/5.7.0/)

**下载**
```
wget http://archive.cloudera.com/cm5/repo-as-tarball/5.7.0/cm5.7.0-centos6.tar.gz
wget http://archive.cloudera.com/cdh5/repo-as-tarball/5.7.0/cdh5.7.0-centos6.tar.gz
wget http://archive.cloudera.com/cdh5/parcels/5.7.0/CDH-5.7.0-1.cdh5.7.0.p0.45-el6.parcel
wget http://archive.cloudera.com/cdh5/parcels/5.7.0/CDH-5.7.0-1.cdh5.7.0.p0.45-el5.parcel.sha1
wget http://archive.cloudera.com/cdh5/parcels/5.7.0/manifest.json
```

**解压**
```
# CM
mkdir -p /var/www/html/cm5/redhat/6/x86_64
tar xvfz cm5.7.0-centos6.tar.gz -C /var/www/html/cm5/redhat/6/x86_64
# CDH
mkdir -p /var/www/html/cdh5
tar xvfz cdh5.7.0-centos6.tar.gz -C /var/www/html/cdh5
mkdir -p /var/www/html/cdh5/parcels/5.7.0/
cp CDH-5.7.0-1.cdh5.7.0.p0.45-el6.parcel CDH-5.7.0-1.cdh5.7.0.p0.45-el6.parcel.sha1  manifest.json /var/www/html/cdh5/parcels/5.7.0/
cd /var/www/html/cdh5/parcels/
ln -s 5.7.0 5.7
ln -s 5.7.0 5
ln -s 5.7.0 latest
```

**其他文件**
```
# CM5
CM5_DIR=/var/www/html/cm5
# CM5 installer
cd $CM5_DIR
wget http://archive.cloudera.com/cm5/installer/5.7.0/cloudera-manager-installer.bin -P installer/5.7.0
cd $CM5_DIR/installer
ln -s 5.7.0 latest

# redhat
REDHAT_DIR=/var/www/html/redhat
mkdir $REDHAT_DIR
cd $REDHAT_DIR

wget http://archive.cloudera.com/redhat/cdh/RPM-GPG-KEY-cloudera -P cdh/
## 或者
ln -s $PWD/../cdh5/redhat/6/x86_64/cdh .
```

**权限**
```
chmod -R ugo+rX /var/www/html/cdh
```


## 其他机器访问
在其他机器添加yum 仓库
```
# cat /etc/yum.repos.d/cloudera.repo
[cloudera]
name=cloudera
baseurl=http://hostname/cm5/redhat/6/x86_64/cm/5
enabled=1
gpgcheck=0 
```

## 参考
* http://www.cloudera.com/documentation/enterprise/latest/topics/cm_ig_create_local_package_repo.html#cmig_topic_21_3





