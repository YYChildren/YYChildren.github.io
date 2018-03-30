---
title: CentOS6 下安装openresty 
date: 2016-08-11 08:53:41
tags:
---


## 安装过程
* 安装依赖库
```
yum install -y readline-devel pcre-devel openssl-devel gcc postgresql-devel
```

* 下载
从<https://openresty.org/cn/download.html>下载一个发行版

* 解压安装到/opt/openresty下
```bash
tar xzvf ngx_openresty-VERSION.tar.gz
cd ngx_openresty-VERSION/
./configure --prefix=/opt/openresty \
            --with-luajit \
            --without-http_redis2_module \
            --with-http_iconv_module \
            --with-http_postgres_module
make -j8
make install
```

* 添加环境变量
```bash
cat >> ~/.bashrc <<EOF
######## openresty #########
export OPENRESTY_HOME=/opt/openresty
export PATH=\$PATH:\$OPENRESTY_HOME/bin
export NGINX_HOME=\$OPENRESTY_HOME/nginx
export PATH=\$PATH:\$NGINX_HOME/sbin
export LUAJIT_HOME=\$OPENRESTY_HOME/luajit
export PATH=\$LUAJIT_HOME/bin:\$PATH
EOF
```

## 参考
* openresty: <https://openresty.org/cn/>
* openresty安装：<https://openresty.org/cn/installation.html>

