# Proxygen开发环境安装

#### 2017-08-01  set_daemon@126.com



## 起因

​	做后台服务首要解决的是寻找一个合适的框架，例如php、java、python做web服务都有相应的至少多于一个成熟服务框架，开发人员只需要了解在框架下如何实现业务逻辑，整个事情就能顺利的拉起往下走，但C++方向的比较轻量级的（niginx/openresty会不会重了些？），找了很久才找到facebook的[proxygen](https://github.com/facebook/proxygen)，不过从网上的资料来看，该框架似乎并不太火，少见有资料来分析，大都简单的描述了下功能，但对我而言，选择它的至关重要的原因是，自己一直在想着有这样一个框架：支持多协议解析、支持（纯）异步、高并发模型、生态较完整、易用、易扩展，尝试自己写过一个框架[asio_fw](https://github.com/set-daemon/asio_fw)，但是并没有实现需求，而且效果也不佳，由于工作原因并没有坚持继续，找到proxygen后，发现设计思路上有一些熟悉，而且似乎也很满足我的需求，决定从这个框架入手，来解答心中的疑惑。



## proxygen依赖及环境

#### 操作系统

​	Ubuntu14（原生支持）、CentOS6.5+（依赖包需要单独安装）

#### gcc

​	版本要求4.9.3+，支持C++14

​	如果系统上版本低于要求，可参考《gcc编译器版本升级》

	修改/etc/ld.so.conf
		/usr/local/lib/c++/4.9.3
		/usr/local/lib64
	执行ldconfig


#### cmake

​	centos yum安装：yum install cmake.x86_64 

​	ubuntu apt安装：sudo apt-get install cmake



#### autoconf

​	centos yum安装：yum install autoconf.noarch

​	ubuntu apt安装：sudo apt-get install autoconf



#### libevent

官网地址：[http://libevent.org/]()

下载最新版本[2017-08-01](https://github.com/libevent/libevent/releases/download/release-2.1.8-stable/libevent-2.1.8-stable.tar.gz)，解压缩后进入执行：

​	./configure

​	make && make install

默认安装路径：/usr/local/lib，如果在/etc/ld.so.conf文件中找不到该路径，则添加进去。

执行ldconfig



#### Boost

官网地址：[http://www.boost.org](http://www.boost.org/)

可以源码安装，也可以使用yum（CentOS）或者apt-get（Ubuntu）。

版本要求：无（官网未说明，暂时用最新[1.64](https://dl.bintray.com/boostorg/release/1.64.0/source/boost_1_64_0.tar.bz2)。）

源码安装：

​	下载源码后，解压后执行./bootstrap.sh

​	生成bjam，可以执行./bjam --show-libraries查询可生成的库列表，proxygen要求所有库

​		atomic、chrono、container、context、coroutine、coroutine2、date_time、exception、fiber、filesystem、graph、graph_parallel、iostreams、locale、log、math、metaparse、mpi、program_options、python、random、regex、serialization、signals、system、test、thread、timer、type_erasure、wave

​	执行./bjam variant=release --build-dir=./build_tmps link=static threading=multi runtime-link=shared --prefix=/usr/local 生成静态和动态、包含所有库的release

#./bjam variant=release link=shared threading=multi runtime-link=shared --build-dir=./build_tmps --build-type=complete --prefix=/usr/local

执行ldconfig



#### double_conversion

下载地址：[https://github.com/google/double-conversion](https://github.com/google/double-conversion)

下载最新发布版本[2017-08-01](https://github.com/google/double-conversion/archive/v2.0.1.tar.gz)，解压后执行

​	cmake . -DBUILD_SHARED_LIBS=ON -DBUILD_STATIC_LIBS=ON -DCMAKE_CXX_COMPILER=/usr/local/bin/g++ -DCMAKE_C_COMPILER=/usr/local/bin/gcc

​	make && make install



##### google glog

下载地址：[[https://github.com/google/glog](https://github.com/google/glog]

下载最新发布版本[2017-08-01](https://github.com/google/glog/archive/v0.3.5.tar.gz)，解压后执行：

​	cmake . -DBUILD_SHARED_LIBS=ON -DBUILD_STATIC_LIBS=ON -DCMAKE_CXX_COMPILER=/usr/local/bin/g++ -DCMAKE_C_COMPILER=/usr/local/bin/gcc

​	make && make install



##### gflags

下载地址：[[https://github.com/gflags/gflags](https://github.com/gflags/gflags)

下载最新版本[2017-08-01](https://github.com/gflags/gflags/archive/v2.2.1.tar.gz),解压缩后进入执行：

​	cmake . -DBUILD_SHARED_LIBS=ON -DBUILD_STATIC_LIBS=ON -DCMAKE_CXX_COMPILER=/usr/local/bin/g++ -DCMAKE_C_COMPILER=/usr/local/bin/gcc

​	make && make install



##### folly

下载地址：[https://github.com/facebook/folly]()

下载最新版本[2017-08-01](https://github.com/facebook/folly/archive/v2017.07.31.00.tar.gz)，解压缩后进入子目录folly执行：

​	autoreconf -ivf

​	./configure --with-boost-libdir=/usr/local/lib

​	make && make install

​	修改/etc/ld.so.conf：

​		/usr/local/lib

​	执行ldconfig以加载最新安装的库，并将库路径设置到LD_LIBRARY_PATH环境变量中。



#### wangle

下载地址：[https://github.com/facebook/wangle]()

下载最新版本[2017-08-01](https://github.com/facebook/wangle/archive/v2017.07.24.00.tar.gz)，解压缩后进入执行：

​	cmake . -DBUILD_SHARED_LIBS=ON -DBUILD_STATIC_LIBS=ON -DCMAKE_CXX_COMPILER=/usr/local/bin/g++ -DCMAKE_C_COMPILER=/usr/local/bin/gcc

​	make && make install



#### 其它库

​	CentOS下：

yum install cmake gcc-c++.x86_64 flex bison.x86_64 krb5-devel.x86_64 cyrus-sasl-devel.x86_64 

numactl-devel.x86_64 libgudev1-devel.x86_64 openssl-devel.x86_64 libcap-devel.x86_64 gperf.x86_64 autoconf.noarch libevent-devel.x86_64 libtool.x86_64 boost-devel.x86_64 gc.x86_64 snappy-devel.x86_64 unzip.x86_64 wget binutils-devel.x86_64

​	Ubuntu下：

sudo apt-get install g++ git cmake flex bison libkrb5-dev libsasl2-dev libnuma-dev pkg-config libssl-dev libcap-dev gperf autoconf-archive libtool libjemalloc-dev libsnappy-dev wget unzip



## Proxygen

下载地址：[[https://](https://github.com/facebook/proxygen)[github.com/facebook/proxygen](https://github.com/facebook/proxygen)]

下载最新版本[2017-08-01](https://github.com/facebook/proxygen/archive/v2017.07.31.00.tar.gz)，解压缩后进入执行：

​	./configure

​	make -j 4 && make install



执行ldconfig
