---
title: Nginx 基础知识
date: 2020-12-03 20:39:51
categories: [learn]
tags: [nginx]
featuredImagePreview: https://nginx.org/nginx.png
---

# Nginx

Nginx 是一款轻量级的 Web 服务器、反向代理服务器及电子邮件代理服务器，其特点是占用内存小，并发能力强。

## Nginx 使用场景

### HTTP 的反向代理服务器

* 正向代理

  ![image-20201203210919188](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/210919.png)

* 反向代理

  ![image-20201203211044613](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/211044.png)

* Nginx 的反向代理

  ![image-20201203211409241](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/211409.png)

* 正向代理：是一个位于客户端和原始服务器之间的服务器，为了从原始服务器取得内容，客户端向代理发送请求并指定原始服务器，然后代理向原始服务器转发请求并将获得的内容返回给客户端。
* 反向代理：代理服务器接收网络上的请求，它将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给原始请求的客户端。

### 动态静态资源分离

* 静态资源无需经过 Tomcat，Tomcat 只负责处理动态请求
* Nginx 本身是一个静态资源服务器

## Nginx 的优点

* 高并发、高性能
* 可扩展性好：模块化设计
* 可靠性高
* 热部署：不停止服务的情况下升级 Nginx
* 开源、可商用

## Nginx 安装使用

### Nginx 在 CentOS7 下的安装

* 安装 yum-utils

  `yum install yum-utils`

* 为 Nginx 增加源

  ```shell
  [root@localhost ~]# vi /etc/yum.repos.d/nginx.repo
  
  [nginx-stable]
  name=nginx stable repo
  baseurl=http://nginx.org/packages/centos/7/$basearch/
  gpgcheck=1
  enabled=1
  gpgkey=https://nginx.org/keys/nginx_signing.key
  module_hotfixes=true
  
  [nginx-mainline]
  name=nginx mainline repo
  baseurl=http://nginx.org/packages/mainline/centos/7/$basearch/
  gpgcheck=1
  enabled=0
  gpgkey=https://nginx.org/keys/nginx_signing.key
  module_hotfixes=true
  ```

* 查看 Nginx 源

  ```shell
  [root@localhost ~]# yum list | grep nginx
  nginx.x86_64                                1:1.18.0-2.el7.ngx         @nginx-stable
  nginx-debug.x86_64                          1:1.8.0-1.el7.ngx          nginx-stable
  nginx-debuginfo.x86_64                      1:1.18.0-2.el7.ngx         nginx-stable
  nginx-module-geoip.x86_64                   1:1.18.0-2.el7.ngx         nginx-stable
  nginx-module-geoip-debuginfo.x86_64         1:1.18.0-2.el7.ngx         nginx-stable
  nginx-module-image-filter.x86_64            1:1.18.0-2.el7.ngx         nginx-stable
  nginx-module-image-filter-debuginfo.x86_64  1:1.18.0-2.el7.ngx         nginx-stable
  nginx-module-njs.x86_64                     1:1.18.0+0.5.0-1.el7.ngx   nginx-stable
  nginx-module-njs-debuginfo.x86_64           1:1.18.0+0.5.0-1.el7.ngx   nginx-stable
  nginx-module-perl.x86_64                    1:1.18.0-2.el7.ngx         nginx-stable
  nginx-module-perl-debuginfo.x86_64          1:1.18.0-2.el7.ngx         nginx-stable
  nginx-module-xslt.x86_64                    1:1.18.0-2.el7.ngx         nginx-stable
  nginx-module-xslt-debuginfo.x86_64          1:1.18.0-2.el7.ngx         nginx-stable
  nginx-nr-agent.noarch                       2.0.0-12.el7.ngx           nginx-stable
  pcp-pmda-nginx.x86_64                       4.3.2-12.el7               base
  ```

* 安装指定版本

  ```shell
  [root@localhost ~]# yum install nginx 1:1.18.0-2.el7.ngx
  ```

* 验证 Nginx 是否安装成功

  ```shell
  [root@localhost ~]# nginx -v
  nginx version: nginx/1.18.0
  ```

* 可能需要配置防火墙

  ```shell
  # 查看防火墙状态
  [root@localhost conf.d]# firewall-cmd --state
  running
  # 查看启用的端口
  [root@localhost conf.d]# firewall-cmd --list-port
  80/tcp
  # 开放 80 端口 --permanent 永久生效
  [root@localhost conf.d]# firewall-cmd --add-port=80/tcp --permanent
  ```

* 或者同时打开 80 和443端口

  ```shell
  firewall-cmd --permanent --zone=public --add-service=http
  firewall-cmd --permanent --zone=public --add-service=https
  firewall-cmd --reload
  ```

### Nginx 常用命令

```shell
[root@localhost ~]# nginx -h
nginx version: nginx/1.18.0
Usage: nginx [-?hvVtTq] [-s signal] [-c filename] [-p prefix] [-g directives]
Options:
  -?,-h         : this help
  -v            : show version and exit
  -V            : show version and configure options then exit
  -t            : test configuration and exit
  -T            : test configuration, dump it and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload
  -p prefix     : set prefix path (default: /etc/nginx/)
  -c filename   : set configuration file (default: /etc/nginx/nginx.conf)
  -g directives : set global directives out of configuration file
```

| 命令     | 含义                     |
| -------- | ------------------------ |
| `?/h`    | 显示帮助信息             |
| `v/V`    | 显示版本信息             |
| `c`      | 使用指定的配置文件       |
| `g`      | 指定配置指令             |
| `p`      | 指定运行目录             |
| `t/T`    | 测试配置文件编写是否正确 |
| `s`      | 向 master 进程发送命令   |
| `stop`   | 立即停止                 |
| `quit`   | 优雅停止                 |
| `reopen` | 重新开始记录日志文件     |
| `reload` | 重启                     |

### Nginx 配置文件

* Nginx 相关的配置文件在`/etc/nginx/`目录中 

* Nginx 的主配置文件是`/etc/nginx/nginx.conf`

* Nginx 日志文件`access.log`和`error.log`位于`/var/log/nginx/`目录下

* 基础语法：

  * 使用`;`结尾
  * 使用`{}`组织多条指令
  * 使用`include`引入其它配置文件
  * 使用`#`注释
  * `$`表示变量

* Nginx 默认配置文件及注释`nginx.conf`

  ```nginx
  user  nginx;
  # 工作进程数
  worker_processes  1;
  
  # Nginx 的错误日志
  error_log  /var/log/nginx/error.log warn;
  # Nginx 的pid，pid 是文本文件，只有一行，记录了该进程的 ID
  pid        /var/run/nginx.pid;
  
  
  events {
      # 每个进程最大连接数
      worker_connections  1024;
  }
  
  
  http {
      include       /etc/nginx/mime.types;
      default_type  application/octet-stream;
      # 自定义日志格式
      log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"';
  
      # 访问日志
      access_log  /var/log/nginx/access.log  main;
  
  
      # 高效传输文件模式
      sendfile        on;
      tcp_nopush     on;
  
      # 请求超时时间
      keepalive_timeout  65;
  
      #gzip  on;
  
      include /etc/nginx/conf.d/*.conf;
  }
  ```

* `default.conf`

  ```nginx
  # 每一个 server 对应着一个虚拟主机或域名
  server {
      # 监听端口
      listen       80;
      # server_name  localhost;
  
      #charset koi8-r;
      #access_log  /var/log/nginx/host.access.log  main;
  
      # 一个 server 可以配置多个 location ，针对不同的访问路径做配置
      # 访问首页
      location / {
          root   /usr/share/nginx/html;
          index  index.html index.htm;
      }
  
      #error_page  404              /404.html;
  
      # redirect server error pages to the static page /50x.html
      #
      # 当出现 5XX 状态码时，重定向到新页面
      error_page   500 502 503 504  /50x.html;
      location = /50x.html {
          root   /usr/share/nginx/html;
      }
  
      # proxy the PHP scripts to Apache listening on 127.0.0.1:80
      #
      #location ~ \.php$ {
      #    proxy_pass   http://127.0.0.1;
      #}
  
      # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
      #
      #location ~ \.php$ {
      #    root           html;
      #    fastcgi_pass   127.0.0.1:9000;
      #    fastcgi_index  index.php;
      #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
      #    include        fastcgi_params;
      #}
  
      # deny access to .htaccess files, if Apache's document root
      # concurs with nginx's one
      #
    #location ~ /\.ht {
      #    deny  all;
      #}
  }
  ```
  
* Nginx 的内置全局变量

  | 变量名             | 描述                                                         |
  | ------------------ | ------------------------------------------------------------ |
  | `$host`            | 请求信息中的 Host，如果请求中没有 Host 行，则等于设置的服务器名 |
  | `$request_method`  | 客户端的请求类型，如 `GET`、`POST`                           |
  | `$remote_addr`     | 客户端的 IP 地址                                             |
  | `$remote_port`     | 客户端的端口                                                 |
  | `$args`            | 请求中的参数                                                 |
  | `$content-length`  | 请求头的 `Content-length` 字段                               |
  | `$http_user_agent` | 客户端 agent 信息                                            |
  | `$http_cookies`    | Cookies 信息                                                 |
  | `$server_protocol` | 请求使用的协议                                               |
  | `$server_address`  | 服务器地址                                                   |
  | `$server_name`     | 服务器名称                                                   |
  | `$server_port`     | 服务器的端口号                                               |

### Nginx 开启压缩

`ngx_http_gzip_module` 为 Nginx 提供了 `gzip` 的压缩支持，可以减少传输资源，节省宽带，但会增加 CPU 消耗。

主要参数有：

| 参数名            | 参数值      | 描述                                                         |
| ----------------- | ----------- | ------------------------------------------------------------ |
| `gzip`            | `on / off`  | 是否开启 `gzip` 压缩                                         |
| `gzip_comp_level` | 1 ~ 9       | 压缩级别                                                     |
| `gzip_min_length` |             | 小于此设定值的文件不会被压缩，长度来源于 `Content-Length` 字段 |
| `gzip_types`      | `text/html` | 进行压缩的 MIME 类型                                         |

### Nginx 配置反向代理

`ngx_http_upstream_module` 为 Nginx 提供列反向代理功能。此模块用于将多个服务器定义为服务器组，可以通过 `proxy_pass` 传递指令来引用服务器组。

一个简单的配置如下：

```nginx
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;
    
    upstream springboot{
        server 127.0.0.1:8080;
        server 127.0.0.1:8081;
        server 127.0.0.1:8082;
    }

    server {
        listen       80;
        server_name  localhost;
        
        location / {
	        proxy_pass http://springboot;
        }
        
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }

    }
}
```

### Nginx 负载均衡策略

upstream 默认依照轮询的方式进行负载均衡，每一个请求按时间顺序逐一分配到不同的后端服务器。假使后端服务器故障，能主动剔除。除此之外，upstream 还有如下的分配策略：

* weight（权重）

  weight 值和分配概率成正比，用于后端服务器性能不均的情况下。

  ```nginx
  upstream springboot{
      server 127.0.0.1:8080 weight=1;
      server 127.0.0.1:8081 weight=2;
      server 127.0.0.1:8082 weight=3;
  }
  ```

* ip_hash（访问 IP）

  每一个请求按访问 IP 的 hash 运算结果分配到指定的后端服务器，能够解决 session 的问题。

  ```nginx
  upstream springboot{
      ip_hash;
      server 127.0.0.1:8080;
      server 127.0.0.1:8081;
      server 127.0.0.1:8082;
  }
  ```

* fair

  根据页面大小、加载时间长短智能的进行负载均衡。

* least_conn（最少连接）

  下一个请求将被分派到活动连接数量最少的服务器。
