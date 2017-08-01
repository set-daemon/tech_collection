# GCC编译器版本升级

#### 2017-08-01  set_daemon@126.com 摘抄整理



总有人走在潮流的前面，创造了诸多强大的工具，当想要使用时发现，坑爹的用了太多新特性，必须在不升级系统的情况下升级编译器版本（致facebook的开源库proxygen）。



## GCC源

https://mirrors.tuna.tsinghua.edu.cn/gnu/gcc/ 清华镜像

https://gcc.gnu.org/mirrors.html



直至现在（2017/8/1），gcc最高版本已经到了7.1.0。究竟选择哪个版本，看依赖库的要求，因为要用proxygen，其版本要求>=4.9，暂时先从4.9开始。

下载gcc4.9.3 https://mirrors.tuna.tsinghua.edu.cn/gnu/gcc/gcc-4.9.3/gcc-4.9.3.tar.bz2。

解压 tar -jxvf gcc-4.9.3.tar.bz2

进入gcc-4.9.3执行./contrib/download_prerequisites脚本，将依赖下载下来，查看脚本可以看到有以下依赖库：

​	mpc-0.8.1.tar.gz

​	gmp-4.3.2.tar.bz2

​	cloog-0.18.1.tar.gz

​	isl-0.12.2.tar.bz2
​	mpfr-2.4.2.tar.bz2

如果是其它gcc版本，对这些依赖库版本也会有要求，做运维的时候需要注意是否可以预先将依赖库下载保留下来，节约每台服务器升级时都需要从远端服务器下载的时间。



在当前目录下创建并进入临时目录 mkdir -p gcc_temp，执行

​	../configure --enable-checking=release --enable-languages=c,c++ --disable-multilib

​	make && make install

编译时间会比较长，可以起来活动下。



## 使用问题



#### 引用问题

由于新升级的版本会默认安装在/usr/local/include/c++，编译应用时引用会出错，可设置环境变量export CPLUS_INCLUDE_PATH=/usr/local/include/c++；

​	如果使用CMake管理项目，应设置cmake使用的g++版本，例如cmake . -DCMAKE_CXX_COMPILER=/usr/local/bin/g++ -DCMAKE_C_COMPILER=/usr/local/bin/gcc

#### 版本冲突

有些库可能支持的gcc版本低，编译过程中会出现错误，第一种方案：修改源码以满足编译要求；第二种方案：切换版本，可参考3用yum的方式安装工具集来管理。



## 说明

已在CentOS6.5系统上测试升级gcc4.9.3成功。



## 参考

1 http://blog.csdn.net/origin_lee/article/details/43231397

2 http://blog.csdn.net/qingfeng_happy7/article/details/49940675

3 http://blog.csdn.net/juweiping/article/details/55194109