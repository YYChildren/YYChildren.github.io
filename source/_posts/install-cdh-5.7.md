---
title: CDH 5.7 安装
date: 2016-05-04 17:54:01
tags:
---

## 搭建镜像
安装位置：192.168.8.209
参考:<http://yychildren.leanote.com/post/安装Cloudera-5.7.0镜像/>
## 准备三台机器
```
ycj1.mingchao.com
ycj2.mingchao.com
ycj3.mingchao.com
```
## 安装基本软件
### 安装yum库
```
yum install -y wget
```
* 163的yum库
```
wget http://mirrors.163.com/.help/CentOS6-Base-163.repo -P /etc/yum.repos.d
```
* epel库
```
rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
```

### 基本软件
```
yum install -y vim lrzsz python openssh openssh-clients
# vim
# lrzsz
# python 2.6/2.7
# openssh
# openssh-clients
```

## host 设置
```
# 你的镜像服务器，参考"安装Cloudera 5.7.0镜像"
192.168.8.209 archive.cloudera.com

192.168.116.21 ycj1.mingchao.com
192.168.116.22 ycj2.mingchao.com
192.168.116.23 ycj3.mingchao.com
```

## 生成ssh密钥对
```
ssh-keygen
```

## 添加公钥
ycj1.mingchao.com上做
```
ssh-copy-id ycj1.mingchao.com
ssh-copy-id ycj2.mingchao.com
ssh-copy-id ycj3.mingchao.com
```

## 禁用selinux
**参考：**http://www.thegeekstuff.com/2009/06/how-to-disable-selinux-redhat-fedora-debian-unix/
**永久**
```
cat /etc/selinux/config
## 之后重启电脑
```
```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#       enforcing - SELinux security policy is enforced.
#       permissive - SELinux prints warnings instead of enforcing.
#       disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of these two values:
#       targeted - Targeted processes are protected,
#       mls - Multi Level Security protection.
SELINUXTYPE=targeted
```
**临时**（没成功）
```
echo 0 > /selinux/enforce
## or
setenforce 0
```
**查看selinux状态**
```
getenforce
# Disabled
```

## 关闭防火墙
**永久（需重启）**
```
chkconfig iptables off
```
**临时**
```
service iptables stop
```

## yum 不检验ssl
**原因是5.7.0的访问以https的形式**
```
# 在/etc/yum.conf 下添加
sslverify=false
```

## 安装Cloudera manager
**ycj1.mingchao.com上做**
```
wget http://archive.cloudera.com/cm5/installer/5.7.0/cloudera-manager-installer.bin
chmod u+x cloudera-manager-installer.bin
# 从Internet安装
sudo ./cloudera-manager-installer.bin
# 从本地库安装 （这里不采用）
sudo ./cloudera-manager-installer.bin --skip_repo_package=1
```
**安装完后等待，直到**
```
netstat -anp | grep 7180
# tcp        0      0 0.0.0.0:7180
# 0.0.0.0:*                   LISTEN      1676/java  
```


## cloudera-manager
### 登录
链接：http://192.168.116.21:7180/
账号：admin
密码：admin

### 下一步到
![](https://leanote.com/api/file/getImage?fileId=587dfeceab644121e3000437)

---
**可以选择使用数据包或者使用Parcel**
![](https://leanote.com/api/file/getImage?fileId=587dfeceab644121e300043f)

---

**使用以下搜索主机：**
```
ycj[1-3].mingchao.com
```

### **继续--安装JDK**
![](https://leanote.com/api/file/getImage?fileId=587dfecfab644121e3000447)

---

选定：
![](https://leanote.com/api/file/getImage?fileId=587dfecfab644121e300044d)

### **继续--SSH登录凭据**
![](https://leanote.com/api/file/getImage?fileId=587dfeceab644121e3000439)

---

选择：
![](https://leanote.com/api/file/getImage?fileId=587dfeceab644121e3000435)

---

![](https://leanote.com/api/file/getImage?fileId=587dfeceab644121e3000443)

### **继续--等待安装**
![](https://leanote.com/api/file/getImage?fileId=587dfeceab644121e3000433)
![](https://leanote.com/api/file/getImage?fileId=587dfeceab644121e3000431)

## 权限问题解决
```
sudo -u hdfs hadoop fs -chmod 777  /
sudo -u hdfs hadoop fs -mkdir -p /user/spark/applicationHistory
sudo -u hdfs hadoop fs -chown -R spark /user/spark
sudo -u hdfs hadoop fs -chmod 1777 /user/spark/applicationHistory
```
### 安装完成



## 证书问题
>#### **报错：**
>![](https://leanote.com/api/file/getImage?fileId=587dfeceab644121e300043b)
>![](https://leanote.com/api/file/getImage?fileId=587dfecfab644121e3000449)
>#### **原因**
>没有cloudera的证书
>#### **解决**
>* 中止安装
>* yum 不验证ssl
>```
># 在/etc/yum.conf 下添加
>sslverify=false
>```
>* 卸载失败的主机
>* 重试失败的主机
>#### **添加证书**
>但在CentOS 6下无效的方法，已经被确定为CentOS的bug：https://www.centos.org/forums/viewtopic.php?t=1073
>**更新ca-certificates**
>```
>yum --disablerepo=cloudera-manager -y update ca-certificates
>```
>**导出cloudera的证书为cloudera.cer**
>![](https://leanote.com/api/file/getImage?fileId=587dfeceab644121e3000445)
>![](https://leanote.com/api/file/getImage?fileId=587dfeceab644121e3000441)
>![](https://leanote.com/api/file/getImage?fileId=587dfeceab644121e300043d)
>![](https://leanote.com/api/file/getImage?fileId=587dfecfab644121e300044b)
>**导入证书**
>```
转换格式 .cer 到 .pem
openssl x509 -inform der -in cloudera.cer  -out cloudera.pem
追加到信任列表
cat cloudera.pem >> /etc/pki/tls/certs/ca-bundle.crt
>```

## 参考
* http://www.cloudera.com/documentation/enterprise/latest/topics/cm_ig_install_path_a.html#cmig_topic_6_5
