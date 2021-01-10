## 基本操作

```shell
#查看版本号
./nginx -v
./nginx -V

#启动

#关闭
./nginx -s stop

#重新加载（配置文件）
./nginx -s reload

```

### CentOS

自带防火墙，需要对外开放端口

```shell
#查看开放的端口号
firewall-cmd --list-all

#设置开放的端口号
firewall-cmd --add-service=http --permanet
sudo firewall-cmd -add-port=80/tcp --pernamet

#重启防火墙
firewall-cmd --reload
```



## 配置文件格式

1. 全局块	：主要设置一些英雄nginx服务器整体运行的配置指令。比如运行nginx服务器的用户（组）、允许生成的worker process数、进程PID存放路径、日志存放路径和类型以及配置文件的导入等
2. events块：对nginx性能影响比较大
3. http块     ：服务器配置中最频繁的部分
   1. http全局块：指令包括文件引入、MIME-TYPE定义、日志自定义、连接超时时间、单连接请求上限等
   2. server块：和虚拟主机有密切关系。虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的



## 反向代理

```shell
#默认
#server{
#	listen 80;
#	server_name localhost;
#
#	location ~ {
#		
#	}
#}

#将监听地址改为代理服务器所在地址
server{
	listen 80;
	server_name 192.168.241.155;
	
	#在location添加转发目的地
	#利用proxy_pass指定目标服务器的ip地址以及服务器运行的端口号
	#location后面的 ~ 代表匹配发送给192.168.241.155:80所有url
	location ~ {
		proxy_pass http://192.168.241.155:8080
	}
	
	#根据正则分类
	#例如访问  192.168.241.155:80/edu将会转发给8080端口
	#location ~ /edu/ {
	#	proxy_pass http://192.168.241.155:8080
	#}
	#例如访问  192.168.241.155:80/vod将会转发给8081端口
	#location ~ /vod/ {
	#	proxy_pass http://192.168.241.155:8081
	#}
}
```



## 负载均衡

```shell
#默认
#http{
#    server{
#       listen 80;
#       server_name localhost;
#		location{
#		}
#    }
#}

http{
	#定义用于负载均衡的服务器及其端口
	upstream myserver{
		server 192.168.241.155:8080;
		server 192.168.241.155:8081;
	}
	
	server{
		listen 80;
		server_name 192.168.241.155;
		location{
			#设置负载均衡服务器
			#客户端发送的请求将会被分发至myserver中设置的服务器中
			proxy_pass http://myserver;
		}
	}
}

#负载均衡的策略
#1.轮询 : 先发送给8080，再发送给8081
#2.weight加权 : 优先给权重高的（先给8081）
#upstream myserver{
#    server 192.168.241.155:8080 weight = 2;
#    server 192.168.241.155:8081 weight = 3;
#}
#3.ip_hash(常用) : 用hash函数对发出请求的ip求值的结果分配，可以确保用户只访问一个服务器
#upstream myserver{
#	 ip_hash;
#    server 192.168.241.155:8080;
#    server 192.168.241.155:8081;
#}
#4.fair(常用) : 根据响应时间进行分配，响应时间短的优先分配
#upstream myserver{
#    server 192.168.241.155:8080;
#    server 192.168.241.155:8081;
#	 fair;
#}
```



## 动静分离

将动态资源（如通过tomcat访问数据库）和静态资源（html页面、css文件等）分开，严格意义上说是将动态请求和静态请求分开

- （主流方法）单纯把静态文件独立成单独的域名，放在独立的服务器上
- 动态跟静态文件混在一起发布，通过nginx分开

实现方法：通过location块指定不同的后缀名实现不同的请求转发



## 高可用

![image-20210111023044869](C:\C++学习资料\学习笔记\keepalived_nginx高可用.png)



### keepalived安装

```shell
#安装依赖(大部分已在nginx安装时安装了)
#ubuntu
apt-get install build-essential
apt-get install libtool
apt install libpcre3 libpcre3-dev
apt install zlib1g-dev
apt install openssl libssl-dev
#centOS
yum -y install gcc automake autoconf libtool make
yum install -y gcc openssl-devel popt-devel

#安装pcre库
cd /usr/local/src  #可更改
wget https://ftp.pcre.org/pub/pcre/pcre-8.44.tar.gz 
tar -zxvf pcre-8.44.tar.gz
cd pcre-8.44
./configure
make && make install
#安装zlib库
wget http://zlib.net/zlib-1.2.11.tar.gz
tar -zxvf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./configure
make && make install
#ssl
wget https://www.openssl.org/source/openssl-1.1.1g.tar.gz
tar -zxvf openssl-1.1.1g.tar.gz
cd openssl-1.1.1g
./config
make && make install
#导入ssl库
ln -s ./libssl.so.1.1 /usr/lib64/libssl.so.1.1
ln -s ./libcrypto.so.1.1 /usr/lib64/libcrypto.so.1.1
#路径可以通过下面命令查找
find / -name libssl.so.1.1

#安装keepalived
#安装自己想要的版本，centOS7必须装1.3.2以上的版本
 wget http://www.keepalived.org/software/keepalived-1.2.17.tar.gz
 tar -zxvf keepalived-1.2.17.tar.gz
 cd keepalived-1.2.17
 #配置时最好加--prefix参数指定安装位置
 ./configure --prefix=/usr/local/keepalived
 make && make install
```



### keepalived配置

```shell
#配置系统启动
mkdir /etc/keepalived
cp /usr/local/keepalived/etc/keepalived/keepalived.conf /etc/keepalived

#1.3以下的版本执行这条
cp /usr/local/keepalived/etc/rc.d/init.d/keepalived /etc/init.d/
#1.3以上的执行这条（具体看在那个文件夹里能找到）
cp /usr/local/src/keepalived-1.2.7/keepalived/etc/init.d/keepalived /etc/init.d/

cp /usr/local/keepalived/etc/sysconfig/keepalived /etc/sysconfig/

#centOS
chkconfig --add keepalived #添加操作系统启动
chkconfig --list keepalived #检查是否添加成功

#ubuntu
#由于除了redhat，其他Linux都没有/etc/rc.d/init.d/functions，需要建立软连接
mkdir -p  /etc/rc.d/init.d
ln -s /lib/lsb/init-functions /etc/rc.d/init.d/functions
#安装daemon，并修改命令
apt install daemon
vim /etc/sysconfig/keepalived
#默认情况下，只有 KEEPALIVED_OPTIONS="-D"生效
#相当于启动keepalived服务时添加-D参数，用来配置日志的
#但在Ubuntu下启动，会替换成 daemon keepalived -D，这样-D会被认为是daemon的参数，会启动失败
#所以需要添加daemon -- keepalived ${KEEPALIVED_OPTIONS}
#相当于 daemon -- keepalive -D
systemctl daemon-reload	#每次修改keepalived后都要重新加载服务脚本

#使用keepalived
systemctl start|status|stop keepalived.service
service keepalived start|status|stop
ip a  #查看虚拟ip信息

```



