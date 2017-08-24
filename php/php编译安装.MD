# PHP编译安装

***set_daemon@126.com 2017-08-22***



| 时间         | 版本   | 说明       |
| ---------- | ---- | -------- |
| 2017-08-22 | V1.0 | php5.6.9 |





## 环境

Linux系统（Centos/Ubuntu）



## 下载

[PHP官网下载](http://php.net/releases/)

[PHP扩展下载网](http://pecl.php.net/)



## 依赖

#### PCRE

CentOS：yum install pcre-devel.x86_64 pcre.x86_64

Ubuntu：



#### libxml

CentOS：yum install libxml2-static.x86_64 libxml++.x86_64 libxml2-devel.x86_64

Ubuntu：



#### zlib

CentOS：yum install zlib-devel.x86_64

Ubuntu：



#### bz2

CentOS：yum install bzip2-devel.x86_64 lbzip2-utils.x86_64 lbzip2.x86_64

Ubuntu：



#### ffmpeg



CentOS：

Ubuntu：



#### curl

CentOS：yum install libcurl.x86_64 libcurl-devel.x86_64

Ubuntu：



#### libmcrypt

CentOS：yum install libmcrypt-devel.x86_64 libmcrypt.x86_64

ubuntu：



#### gd

CentOS：yum install gd-devel.x86_64 



## 编译安装

###### 编译安装php

./configure --prefix=/usr/local/php --with-zlib --enable-mbstring --with-openssl --with-mysql --with-mysqli --with-mysql-sock --with-gd --with-jpeg-dir=/usr/lib --enable-gd-native-ttf  --enable-pdo --with-pdo-mysql --with-gettext --with-curl --with-pdo-mysql --enable-sockets --enable-bcmath --enable-xml --with-bz2 --enable-zip --enable-freetype --enable-fpm 

make &&make install

修改/etc/bashrc并执行. /etc/bashrc：

​	export PHP_HOME=/usr/local/php

​	export PATH=\${PHP_HOME}/sbin:\${PHP_HOME}/bin:\${PATH}

###### 编译安装扩展mongodb

进入mongo扩展[下载页面](http://pecl.php.net/get/mongodb-1.2.9.tgz)以及[页面](http://pecl.php.net/package/mongo)

解压后进入，执行以下操作：

​	phpize

​	./configure --with-php-config=/usr/local/php/bin/php-config

​	make && make install

###### 编译安装redis

进入redis扩展[下载页面](http://pecl.php.net/package/redis)

解压后进入，执行以下操作：

​	phpize

​	./configure --with-php-config=/usr/local/php/bin/php-config

​	make && make install



###### 编译安装ftp

进入php源码包下的ext/ftp目录下，执行以下操作：

​	phpize

​	./configure --with-php-config=/usr/local/php/bin/php-config

​	make && make install



######编译安装pdo_mysql

进入php源码包下的ext/pdo_mysql目录下，执行以下操作：

​	phpize

​	./configure --with-php-config=/usr/local/php/bin/php-config

​	make && make install

###### 编译安装mcrypt

进入php源码包下的ext/mcrypt目录下，执行以下操作：

​	phpize

​	./configure --with-php-config=/usr/local/php/bin/php-config

​	make && make install



## 配置启动

#### php-fpm配置启动

修改配置文件${PHP_HOME}/etc/php-fpm.conf：

​	

启动：php-fpm -D -c \${PHP_HOME}/etc/php.ini -y \${PHP_HOME}/etc/php-fpm.conf



## 参考

http://www.cnblogs.com/alexqdh/archive/2012/11/20/2776017.html

