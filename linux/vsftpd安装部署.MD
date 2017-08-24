# vsftpd安装部署

***set_daemon@126.com    2017-08-24***



## 环境

Linux系统（CentOS6.5+）；



## 安装

CentOS：yum  install vsftpd.x86_64



## 配置

修改配置文件/etc/vsftpd/vsftpd.conf：

​	anonymous_enable=NO
​	local_enable=YES
​	write_enable=YES
​	local_umask=022
​	anon_upload_enable=FALSE
​	anon_mkdir_write_enable=FALSE
​	dirmessage_enable=YES
​	xferlog_enable=YES
​	connect_from_port_20=YES
​	xferlog_std_format=YES
​	idle_session_timeout=600
​	data_connection_timeout=120
​	ascii_upload_enable=YES
​	ascii_download_enable=YES
​	ftpd_banner=Welcome to lightnear FTP service.
​	chroot_local_user=YES
​	ls_recurse_enable=YES
​	listen=YES
​	pam_service_name=vsftpd
​	userlist_enable=YES
​	userlist_deny=NO
​	local_root=/data/ftpdata
​	tcp_wrappers=YES
​	use_localtime=YES
​	pasv_enable=YES
​	pasv_min_port=10000
​	pasv_max_port=20000
​	allow_writeable_chroot=YES



添加ftp系统用户：

​	useradd ftper

​	passwd ftper

将用户ftper添加到/etc/vsftpd/user_list文件中。

修改/data/ftpdata的权限所属：

​	chown ftper /data/ftpdata -R

如果有多个用户，则可以创建一个专门用于ftp的用户组，并设置用户所属该组。

##启动

CentOS：service vsftpd start



## 参考

http://jingyan.baidu.com/article/380abd0a77ae041d90192cf4.html

http://www.linuxidc.com/Linux/2015-01/111964.htm