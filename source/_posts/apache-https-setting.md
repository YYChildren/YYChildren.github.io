---
title: apache 的HTTPS设置
date: 2016-05-05 16:39:30
tags:
---


## 配置Apache的HTTP支持
* 创建并切换到ssl目录
```
mkdir -p /etc/httpd/ssl
chmod 600 /etc/httpd/ssl
cd /etc/httpd/ssl
```
* 生成证书和密钥
```
# 建立服务器密钥
openssl genrsa -out server.key 1024
# 建立服务器公钥 
openssl req -new -key server.key -out server.csr
# 建立服务器证书 
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```
* 修改/etc/httpd/conf.d/ssl.conf
```
Listen Server_FQDN:443
SSLEngine on
SSLCertificateKeyFile /etc/httpd/ssl/server.key
SSLCertificateFile /etc/httpd/ssl/server.crt
SSLCACertificateFile /etc/pki/tls/certs/ca-bundle.crt
SSLCipherSuite ALL:-ADH:+HIGH:+MEDIUM:-LOW:-SSLv2:-EXP
```

## 参考：

* http://httpd.apache.org/docs/2.2/
* http://security-24-7.com/how-to-implement-ssl-on-apache-2-2-15/
* http://www.thinksaas.cn/topics/0/280/280017.html