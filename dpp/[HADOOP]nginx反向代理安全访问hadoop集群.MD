# nginx反向代理安全访问hadoop集群
***set_daemon@126.com 2018-04-26***

## 需求
1. 限制公网直接访问hadoop集群（默认的web访问hdfs和resourceManager没有权限管理）
2. 代理访问，修改访问端口，增加权限验证

## nginx方案
### 依赖
1. 第三方模块http://code.google.com/p/substitutions4nginx/，其git目录(ngx_http_substitutions_filter_module)[git://github.com/yaoweibin/ngx_http_substitutions_filter_module.git]。

### 下载nginx、编译安装
假设已经下载nginx及其依赖。
进入到nginx源码解压目录，执行`./configure --prefix=/usr/local/nginx --pid-path=/usr/local/nginx.pid  --with-http_dav_module --with-http_flv_module --with-http_realip_module --with-http_gzip_static_module --with-http_stub_status_module  --with-debug --add-module=../ngx_http_substitutions_filter_module/`。
执行 make && make install。

### 配置nginx
1. conf/nginx.conf

2. conf/conf.d/hadoop.conf
```
upstream master {
        server 127.0.0.1:50070;
}

upstream resmanager {
        server 127.0.0.1:8042;
}

# namenode
server {
        listen 30008;

        location / {
        		auth_basic "authetication for hadoop";
        		auth_basic_user_file hadoop_user_pwd;
                proxy_pass http://master;
                subs_filter_types text/html text/css text/xml;
                subs_filter 127.0.0.1:50075 127.0.0.1:8002/hd11;
                #subs_filter 127.0.0.1:50075 127.0.0.1:8002/hd12;
        }

        location /hd11 {
                proxy_pass http://127.0.0.1:50075/;
        }
}

# resourcemanager
server {
        listen 30018;

        location / {
        		auth_basic "authetication for hadoop";
        		auth_basic_user_file hadoop_user_pwd;
                proxy_pass http://resmanager;
                subs_filter_types text/html text/css text/xml;
                subs_filter 127.0.0.1:50060 127.0.0.1:8001/hd11;
                #subs_filter 127.0.0.1:50060 127.0.0.1:8001/hd12;
        }

        location /hd11 {
                proxy_pass http://127.0.0.1:50060/tasktracker.jsp;
        }
}
```

3. conf/hadoop_user_pwd
创建用户名/密码文件，执行`printf "admin:$(openssl passwd -crypt xyz12345)\n" >>conf/hadoop_user_pwd`，则可以生成用户名admin，密码xyz12345的记录。


### 修改hadoop配置
1. etc/hadoop/hdfs-site.xml
增加配置：
```
	<property>
        <name>dfs.namenode.http-address</name>
        <value>localhost:50070</value>
    </property>
```

2. etc/hadoop/yarn-site.xml
增加/修改配置：
```
	<property>
        <name>yarn.resourcemanager.address</name>
        <value>localhost:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>localhost:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>localhost:8031</value>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value>localhost:8033</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>localhost:8088</value>
    </property>

    <property>
        <name>yarn.nodemanager.webapp.address</name>
        <value>localhost:8042</value>
    </property>

```

## 参考
1. [通过nginx实现内网hadoop、hbase集群对外访问web界面](http://blackwing.iteye.com/blog/1949154)
2. [nginx用户认证配置](http://www.ttlsa.com/nginx/nginx-basic-http-authentication/)
3. [nginx添加认证](https://www.cnblogs.com/cuishuai/p/7761031.html)