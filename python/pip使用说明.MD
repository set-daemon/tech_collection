# pip使用说明

***set_daemon@126.com 2017-09-22***

使用默认的pip源时，有时会出现超时或者https访问失败的问题，还好国内有大量的镜像，感谢这些组织！

##设置源

注意：Ubuntu新版本需要https源。

| 源      | 源地址                                      |
| ------ | ---------------------------------------- |
| 清华     | https://pypi.tuna.tsinghua.edu.cn/simple |
| 阿里云    | http://mirrors.aliyun.com/pypi/simple/   |
| 中国科技大学 | https://pypi.mirrors.ustc.edu.cn/simple/ |
| 华中理工大学 | http://pypi.hustunique.com/              |
| 山东理工大学 | http://pypi.sdutlinux.org/               |
| 豆瓣     | http://pypi.douban.com/simple/           |



临时使用：pip {操作}  -i 源

配置后长期有效：

​	windows下，在用户目录下创建pip目录，再创建pip.ini文件

​	Linux下，修改~/.pip/pip.conf文件

​	文件内容：

​	[global]

​	index-url = https://pypi.tuna.tsinghua.edu.cn/simple
​	[install]

​	trusted-host=mirrors.aliyun.com



##超时/重试

在pip命令后增加选项 --timeout 超时秒数  --retries 重试次数。





## 参考

http://www.cnblogs.com/microman/p/6107879.html 