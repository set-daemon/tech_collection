# maven镜像

***set_daemon@126.com 2017-09-22***

maven管理项目比较方便，但是在国内使用起来问题暴露出来了，依赖包并非系统自带的，所以每次都需要下载，而下载源是国外的话，毕竟是资本主义与共产主义国家有别，这速度跟不上，最为痛苦。还好国内一些组织有为IT界贡献力量的心，将maven源做了镜像，赞！



##镜像源

| 源    | 地址                                       | 说明   |
| ---- | ---------------------------------------- | ---- |
| 阿里云  | http://maven.aliyun.com/nexus/content/groups/public/ |      |



##修改maven镜像配置



#### 远程

修改~/.m2或${maven.home}/conf或者${user.home}/.m2文件夹下的settings.xml文件：

\<?xml version="1.0" encoding="UTF-8"?>
\<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"

          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <mirrors>
        <!-- 阿里云仓库 -->
        <mirror>
            <id>alimaven</id>
            <mirrorOf>central</mirrorOf>
            <name>aliyun maven</name>
            <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
        </mirror>
    
        <!-- 中央仓库1 -->
        <mirror>
            <id>repo1</id>
            <mirrorOf>central</mirrorOf>
            <name>Human Readable Name for this Mirror.</name>
            <url>http://repo1.maven.org/maven2/</url>
        </mirror>
    
        <!-- 中央仓库2 -->
        <mirror>
            <id>repo2</id>
            <mirrorOf>central</mirrorOf>
            <name>Human Readable Name for this Mirror.</name>
            <url>http://repo2.maven.org/maven2/</url>
        </mirror>
    </mirrors> 
\</settings>



#### 本地

修改~/.m2或\${maven.home}/conf或者\${user.home}/.m2文件夹下的settings.xml文件：

\<settings>

​	\<localRepository>\${repository_path}\</localRepository>

\</settings>



#### 私服

使用nexus搭建私服，软件下载https://www.sonatype.com。



## 参考

http://www.cnblogs.com/xunianchong/p/5684042.html

http://www.cnblogs.com/duking1991/p/6110192.html

http://blog.csdn.net/wanghantong/article/details/36427433

http://blog.csdn.net/mydeman/article/details/5905424 用Nexus搭建私服