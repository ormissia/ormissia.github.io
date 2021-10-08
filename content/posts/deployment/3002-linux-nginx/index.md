---
title: Linux部署Nginx流程
date: 2021-10-08T14:18:39+08:00
hero: head.svg
menu:
  sidebar:
    name: Linux部署Nginx流程
    identifier: deployment-linux-nginx
    parent: deployment
    weight: 3002
---

## 安装依赖

### 编译工具及库文件

```shell
yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel
```

### 安装PCRE

> `PCRE`作用是让`Nginx`支持`Rewrite`功能

`PCRE`安装包下载地址：
[https://sourceforge.net/projects/pcre/files/pcre/](https://sourceforge.net/projects/pcre/files/pcre/)

选择对应版本下载即可

#### 下载PCRE安装包

```shell
cd /usr/local/src/
wget http://downloads.sourceforge.net/project/pcre/pcre/8.45/pcre-8.45.tar.gz
```

#### 解压安装包并进入目录

```shell
tar -zxvf pcre-8.45.tar.gz
cd pcre-8.45
```

#### 编译安装

```shell
./configure
make && make install
```

#### 验证安装

```shell
pcre-config --version
```

#### 可能遇到的问题

安装完成之后有可能找不到命令，查看编译安装时的默认安装目录，将其添加到`Linux`环境变量`PATH`即可

## 创建管理Nginx的用户和组

> 创建`nginx`运行用户`nginx`并加入到`nginx`组，不允许`nginx`用户直接登录系统

```shell
groupadd nginx
useradd -g nginx nginx -s /sbin/nologin
```

## 安装Nginx

### 下载安装包

`Nginx`下载地址：
[http://nginx.org/en/download.html](http://nginx.org/en/download.html)

没有特殊需求的话，选择`Stable version`稳定版下载即可

```shell
cd /usr/local/src/
wget http://nginx.org/download/nginx-1.20.1.tar.gz
```

### 解压安装包并进入目录

```shell
tar -zxvf nginx-1.20.1
cd nginx-1.20.1
```

### 编译安装

```shell
./configure \
--prefix=/usr/local/nginx \
--user=nginx --group=nginx \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi \
--with-http_stub_status_module \
--with-http_ssl_module \
--with-http_realip_module

make && make install
```

## 优化Nginx程序的执行路径

添加软连接到环境变量`PATH`目录下

```shell
ln -s /usr/local/nginx/sbin/nginx /usr/local/sbin/
```

添加软连接之后即可直接使用`nginx`命令启动操作`Nginx`

```shell
nginx
nginx -s reload #重启（修改配置文件后重新加载等）
nginx -s quit   #退出（处理完所有请求后结束进程）
nginx -s stop   #停止（直接结束进程）
```

## 测试安装

```shell
nginx -t
```

如果报错，一般是缺少配置路径中的文件夹，使用`mkdir -p`创建即可

修改之后，正常的提示为：

```shell
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```

## 配置开机启动

### 创建启动脚本

```shell
vi /etc/init.d/nginx
```

脚本内容：

> 我这里是修改过的，也可以去官网复制。[官网脚本连接](https://www.nginx.com/resources/wiki/start/topics/examples/redhatnginxinit/)

要注意从官网直接复制的需要修改几处地方，否则运行报错。  
1. nginx变量的值要改成nginx程序的实际安装路径
2. NGINX_CONF_FILE变量的值要改成nginx配置文件的路径
3. lockfile变量的值要改成实际的lockfile文件路径

```shell
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description:  NGINX is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /etc/nginx/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /var/run/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

nginx="/usr/local/nginx/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"

[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx

lockfile=/var/lock/nginx.lock

make_dirs() {
   # make required directories
   user=`$nginx -V 2>&1 | grep "configure arguments:.*--user=" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   if [ -n "$user" ]; then
      if [ -z "`grep $user /etc/passwd`" ]; then
         useradd -M -s /bin/nologin $user
      fi
      options=`$nginx -V 2>&1 | grep 'configure arguments:'`
      for opt in $options; do
          if [ `echo $opt | grep '.*-temp-path'` ]; then
              value=`echo $opt | cut -d "=" -f 2`
              if [ ! -d "$value" ]; then
                  # echo "creating" $value
                  mkdir -p $value && chown -R $user $value
              fi
          fi
       done
    fi
}

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    sleep 1
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $prog -HUP
    retval=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```

### 修改脚本权限

```shell
cd /etc/init.d/
chmod 755 nginx 
```

> 这个脚本可用来直接操作Nginx，直接执行脚本会提示需要输入

### 将Nginx加入到系统服务中

```shell
chkconfig --add nginx
```

### 设为开机启动

```shell
chkconfig nginx on
```

> 重启系统后生效

重启后，即可使用`systemctl`命令管理`nginx`服务

```shell
systemctl status nginx.service
systemctl start nginx.service
systemctl stop nginx.service
```

### 可能遇到的问题

如果使用    命令查看`nginx`服务状态时的提示：
```shell
● nginx.service - SYSV: NGINX is an HTTP(S) server, HTTP(S) reverse proxy and IMAP/POP3 proxy server
   Loaded: loaded (/etc/rc.d/init.d/nginx; generated)
   Active: inactive (dead)
     Docs: man:systemd-sysv-generator(8)
```

是因为系统安装了`httpd`，卸载即可

```shell
yum remove httpd -y
```

## 验证安装

执行

```shell
curl 127.0.0.1
```

从返回结果中可以看到，成功拿到`Nginx`的默认页面了，安装成功  
也可以在外网通过服务IP访问，需要注意Linux防火墙、云服务器出站入站规则等限制

```shell
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
