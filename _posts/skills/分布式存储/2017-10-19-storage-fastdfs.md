---
layout: post
title: "FastDfs 安装配置"
date: 2017-10-19 11:35:00 +0800 
categories: FastDFS
tag: FastDFS
---
* content
{:toc}

FastDFS 安装配置，以及使用说明。

<!-- more -->

## 1 服务器准备
10.4.125.172，10.4.125.173，10.4.125.174，10.4.125.175四台服务器<br/>
其中10.4.125.172，10.4.125.173两台作为Tracker Server,10.4.125.174，10.4.125.175两台作为Storage Server。<br/>

## 2 软件安装
1，FastDFS fastdfs-master.zip 版本5.0.11<br/>
2，libfastcommon-master.zip 下载最新版本即可。<br/>
3，Nginx nginx-1.13.6.tar.gz<br/>
4，fastdfs-nginx-module-master.zip<br/>
5，libevent-2.0.22-stable.tar.gz<br/>

在所有节点安装libfastcommon，安装FastDFS
```
unzip libfastcommon-master.zip
cd libfastcommon-master
./make.sh
./make.sh install

```
   
```
unzip fastdfs-master.zip
cd fastdfs-master
./make.sh
./make.sh install

```

在存储节点安装Nginx与fastdfs-nginx-module

```
安装libevent
tar zxvf libevent-2.0.22-stable.tar.gz
cd libevent-2.0.22-stable
./configure
make & make install

安装nginx
unzip fastdfs-nginx-module-master.zip
tar zxvf nginx-1.13.6.tar.gz
cd nginx-1.13.6
./configure --prefix=/home/app/nginx --add-module=../fastdfs-nginx-module-master/src
make & make install

```

## 3 配置信息

一、配置Tracker 安装后配置文件在系统 /etc/fdfs目录，172和173两台Tracker Server都需要进行配置
```
cd /etc/fdfs
cp tracker.conf.sample tracker.conf
编辑tracker.conf文件，主要修改一些几个参数
（1）disable=false #是否启用，默认为false不需要修改
（2）bind_addr #绑定的IP地址，不写为绑定机器的所有IP
（3）port = 22122 #如果机器该端口被占用需要更换端口
（4）base_path=/home/fdfs/fastdfs_tracker #配置data和log信息的路径自己建立存储路径即可
（5）store_lookup=2 #可以配置0，1，2三个值，0位循环轮询，1为制定group，2为load balance
（6）store_group=group_name # store_lookup为1时必须设置group_name

```


二、配置Stroage 174,175两台机器进行配置
```
cd /etc/fdfs
cp storage.conf.sample storage.conf
编辑storage.conf
（1）disable=false #是否启用，默认为false不需要修改
（2）bind_addr #绑定的IP地址，不写为绑定机器的所有IP
（3）port = 23000 #如果机器该端口被占用需要更换端口
（4）store_path_count=1 #配置存储节点个数
（5）store_path0=/home/fdfs/fastdfs_storage #配置存储节点路径，若store_path_count>1需要配置多个store_path ，例如store_paht1,store_path2
（6）tracker_server=10.4.125.172:22122 <br />
    tracker_server=10.4.125.173:22122 #配置几个tracker server 就写几个
（7）allow_hosts=* #允许访问的IP列表，*为所有IP
（8）http.server_port=8889 #storage的web服务端口
```

三、配置存储节点的nginx
```
（1）将fastdfs-master文件夹中conf目录下的http.conf,mime.types复制到/etc/fdfs目录中<br/>
	 cp http.conf mime.types /etc/fdfs/
（2）将fastdfs-nginx-module-master文件夹下src目录中的mod_fastdfs.conf 复制到/etc/fdfs目录中<br/>
     cp mod_fastdfs.conf /etc/fdfs/
（3）配置mode_fastdfs.conf 文件，需要配置的主要参数<br/>
	 base_path=/home/fdfs/fastdfs_storage # log日志目录
	 load_fdfs_parameters_from_tracker=true # 是否从tracker获取server端的配置信息，默认为false
	 tracker_server=10.4.125.172:22122
	 tracker_server=10.4.125.173:22122 #设置tracker server
	 storage_server_port=23000 #设置为本服务器的storage server端口
	 group_name=group1 #设置本地的storage server group名称
	 url_have_group_name = true #文件路径是否包含group名名称，true为包含/group1/M00/00/01/xxxxxxxx.jpg
	 store_path_count=1 #本地storage 存储路径个数
	 store_path0=/home/fdfs/fastdfs_storage_data #根据store_path_count值设置，有几个写几个，store_path1......
	 group_count=2 #storage server 设置的group数量，有几个后边就需要配置几组group信息
	 [group1]
	 group_name=group1
	 storage_server_port=23005
	 store_path_count=1
	 store_path0=/home/fdfs/fastdfs_storage_data
	 [group2]
	 group_name=group2
	 storage_server_port=23005
	 store_path_count=1
	 store_path0=/home/fdfs/fastdfs_storage_data
```

四、配置Storage Server上的Nginx，主要配置的参数如下
```
（1）listen 8889; #storage.conf 配置的web服务端口
（2）添加一个location 配置
	location ~/group([0-9])/M00 {
            ngx_fastdfs_module;
    }
```

## 4 启动服务 

一、启动tracker server
```
ln -s /usr/bin/fdfs_trackerd /user/local/bin/fdfs_trackerd

fdfs_tracker /etc/fdfs/tracker.conf

ps -ef | grep fdfs* #查看进程是否存在

netstat -an | grep 22122 #查看端口状态

```

二、启动storage server
```
ln -s /usr/bin/fdfs_storaged /usr/local/bin/fdfs_storaged

fdfs_storaged /etc/fdfs/storage.conf

ps -ef | grep fdfs* #查看进程

netstat -an | grep 23000

```
   
三、启动storage 上的nginx
```
/home/app/nginx/sbin/nginx & #启动
```

四、测试上传
```
在tracker server上编辑 client.conf
cd /etc/fdfs
cp client.conf.sample client.conf
vi client.conf
主要编写修改一些内容:
（1）base_path=/home/fdfs/fastdfs_tracker
（2）tracker_server=10.4.125.172:22122
    tracker_server=10.4.125.173:22122  #tracker server地址，端口
（3）http.tracker_server_port=8888 #配置tracker server的web端口地址

保存退出

fdfs_upload_file /etc/fdfs/client.conf AppIcon180.png
返回：group1/M00/00/01/CgR9rlnxj9eAPRFEAAAcGAyzICM808.png
说明上传成功

浏览器中查看：
http://10.4.125.174:8889/group1/M00/00/01/CgR9rlnxj9eAPRFEAAAcGAyzICM808.png
出现图片说明上传成功。

```

## 5 说明
（1）最好一个group配置多个storage 节点，上传到group后所有的storage节点都会保存文件的副本，这样防止文件丢失。
（2）可以采用Nginx为存储节点作负载均衡以及缓存，示例配置信息如下
```
worker_processes  2;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  65535;
    use epoll;
}
http{
	server_names_hash_bucket_size 128;
    client_header_buffer_size 32k;
    large_client_header_buffers 4 32k;
    client_max_body_size 300m;
    sendfile        on;
    tcp_nopush      on;
    proxy_redirect off;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_connect_timeout 90;
    proxy_send_timeout 90;
    proxy_read_timeout 90;
    proxy_buffer_size 16k;
    proxy_buffers 4 64k;
    proxy_busy_buffers_size 128k;
    proxy_temp_file_write_size 128k;
    #设置缓存存储路径、存储方式、分配内存大小、磁盘最大空间、缓存期限
    proxy_cache_path /var/cache/nginx/proxy_cache levels=1:2 keys_zone=http-cache:500m max_size=10g inactive=30d;
    proxy_temp_path /var/cache/nginx/proxy_cache/tmp;

    upstream fdfs_group1{
        server 10.4.125.174:8889 weight=1 max_fails=2 fail_timeout=30s;
    }
    upstream fdfs_group2{
        server 10.4.125.175:8889 weight=1 max_fails=2 fail_timeout=30s;
    }
	
	server {
		listen       8889;
        server_name  localhost;
        location /group1/M00 {
            proxy_next_upstream http_502 http_504 error timeout invalid_header;
            proxy_cache http-cache;
            proxy_cache_valid  200 304 12h;
            proxy_cache_key $uri$is_args$args;
            proxy_pass http://fdfs_group1;
            expires 30d;
        }

        location /group2/M00 {
            proxy_next_upstream http_502 http_504 error timeout invalid_header;
            proxy_cache http-cache;
            proxy_cache_valid  200 304 12h;
            proxy_cache_key $uri$is_args$args;
            proxy_pass http://fdfs_group2;
            expires 30d;
        }
        location ~ /purge/(/.*) {
            allow 127.0.0.1;
            allow 172.16.1.0/24;
            deny all;
            proxy_cache_purge http-cache $1$is_args$args;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
	}

}
```
（3）后续开发客户端程序，支持上传，下载，删除等操作。<br/>
下载fastdfs-client-java-master.zip Maven工程，打包jar，导入到web工程进行后续开发工作。

