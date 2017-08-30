#GO语言学习笔记

***set_daemon@126.com 2017-08-30***



##学习目的

寻找解决高并发服务开发新的解决方案；

寻找更稳健、易于使用、提升开发效率的语言；

调查语言的生态环境完整性，可以解决哪些特定领域的问题；



##与其它语言比较

#### C++

#### Java

#### Javascript

####PHP

#### Python



##生态相关

####nginx



#### redis



#### mongodb



#### kafka



#### mysql



####hdfs



#### hive



#### hbase



##特性/库

参考 https://golang.org/pkg/

####标准库



| 库名        | 子模块             | 摘要                                       |
| --------- | --------------- | ---------------------------------------- |
| archive   | tar             | 访问tar包                                   |
|           | zip             | 支持读写ZIP包                                 |
| bufio     | -               | 待缓存的I/O，包装了io.Reader和io.Writer对象         |
| builtin   | -               | 为Go预声明标志符提供文档                            |
| bytes     | -               | 提供字节分片操作函数                               |
| compress  | bzip2           | 实现bzip2解压                                |
|           | flate           | 支持DEFLATE压缩数据格式，RFC1951                  |
|           | gzip            | 支持gzip格式压缩文件的读写，RFC1952                  |
|           | lzw             | 实现Lempel-Ziv-Welch压缩数据格式，参考T. A. Welch, "A Technique for High-Performance Data Compression", Computer, 17(6)(June 1984),pp 8-19 |
| container | heap            | 支持实现了heap接口的任意类型调用heap操作                 |
|           | list            | 实现双向链表                                   |
|           | ring            | 实现了循环列表的操作                               |
| context   | -               | 定义Context类型，携带deadlines、取消信号以及其它request-scoped值跨API边界及进程 |
| crypto    | aes             | 实现AES加密                                  |
|           | cipher          | 实现标准block cipher模式                       |
|           | des             | 实现Data Encryption Standard(DES)和Triple Data Encryption Algorithm(TDEA) |
|           | dsa             | Digital Signature Algorithm, FIPS 186-3  |
|           | ecdsa           | Elliptic Curve Digital Signature Algorithm, FIPS 186-3 |
|           | elliptic        |                                          |
|           | hmac            | Keyed-Hash Message Authentication Code(HMAC) |
|           | md5             | MD5 hash算法，RFC1321                       |
|           | rand            | 加密安全性伪随机数字生成器                            |
|           | rc4             | RC4加密，Bruce Schneier's Applied Cryptography |
|           | rsa             | RSA加密，                                   |
|           | sha1            | SHA-1哈希算法， RFC3174                       |
|           | sha256          | 实现SHA224和SHA256哈希算法                      |
|           | sha512          | SHA-384、SHA-512/224、SHA-512/256，FIPS 180-4 |
|           | subtle          |                                          |
|           | tls             | TLS 1.2，RFC 5246                         |
|           | x509            | X.509-编码 密钥和认证                           |
|           | pkix            | ASN.1解析，X.509认证序列化，CRL和OCSP              |
| database  | sql             | SQL数据库通用接口                               |
|           | driver          | 定义由包sql使用的数据库sql包实现的接口                   |
| debug     | dwarf           |                                          |
|           | elf             |                                          |
|           | gosym           |                                          |
|           | macho           |                                          |
|           | pe              |                                          |
|           | plan9obj        |                                          |
| encoding  | ascii85         |                                          |
|           | asn1            |                                          |
|           | base32          |                                          |
|           | base64          |                                          |
|           | binary          |                                          |
|           | csv             | 读写以逗号分隔数据的文件                             |
|           | gob             |                                          |
|           | hex             |                                          |
|           | json            |                                          |
|           | pem             |                                          |
|           | xml             |                                          |
| errors    | -               | 实现操作错误的函数                                |
| expvar    | -               | 为公共变量提供标准化接口                             |
| flag      | -               | 命令行标识解析                                  |
| fmt       | -               | 提供类似C语言printf和scanf的I/O格式化函数             |
| go        | ast             | 为Go包声明用于代表语法树的类型                         |
|           | build           | 收集有关Go包的信息                               |
|           | constant        |                                          |
|           | doc             |                                          |
|           | format          | 标准的Go源格式化                                |
|           | importer        |                                          |
|           | parser          | Go源文件的解析器                                |
|           | printer         | AST节点打印                                  |
|           | scanner         | Go源文本的扫描器                                |
|           | token           |                                          |
|           | types           |                                          |
| hash      | adler32         |                                          |
|           | crc32           |                                          |
|           | crc64           |                                          |
|           | fnv             |                                          |
| html      | -               |                                          |
|           | template        | 实现了防代码注入的生成html输出的数据驱动型模板器               |
| image     |                 |                                          |
|           | color           |                                          |
|           | palette         |                                          |
|           | draw            |                                          |
|           | gif             |                                          |
|           | jpeg            |                                          |
|           | png             |                                          |
| index     | suffixarray     | 实现对数时间内的子串搜索内存型后缀数组                      |
| io        | -               |                                          |
|           | ioutil          |                                          |
| log       |                 |                                          |
|           | syslog          |                                          |
| math      |                 |                                          |
|           | big             |                                          |
|           | bits            |                                          |
|           | cmplx           |                                          |
|           | rand            |                                          |
| mime      |                 |                                          |
|           | multipart       |                                          |
|           | quotedprintable |                                          |
|           |                 |                                          |







##go开发环境搭建

go语言官网下载链接：https://golang.org/dl/。

1 预编译安装

​	下载二进制包，解压到预定义目录下即可。

2 源码安装

​	

在上述工作完成后，设置环境变量

​	***linux***下，修改/etc/profile：

​		export GOROOT=/usr/local/go

​		export PATH=\${GOROOT}/bin:\${PATH}

​	执行. /etc/profile

​	***windows***下，



GO编辑器参考https://github.com/golang/go/wiki/IDEsAndTextEditorPlugins



##参考

http://www.runoob.com/go/go-tutorial.html

https://studygolang.com/articles/2620

https://golang.org/doc/

