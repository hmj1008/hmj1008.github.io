---
title: Linux(centOS 7/ubuntu 16)部署django项目
date: 2017-08-17 19:53:05
toc: true
tags:
    - 代码
    - linux
    - 部署
---

KeyWord: ***Linux, Python, Virtualenv, Nginx, Uwsgi, Postgresql, Redis pip,*** 

<!-- more -->
## 1.python2.7 环境下搭建 python3.x 环境
```bash
//安装 pip 
yum install python-setuptools 
easy_install pip 
//安装virtualenv 
pip install virtualenv
```
## 2.在centOS 7 下安装python3.4.3 or py3.X
```bash
mkdir python343_tgz
cd python343_tgz
wget https://www.python.org/ftp/python/3.4.3/Python-3.4.3.tgz
tar -xzf Python-3.4.3.tgz
cd Python-3.4.3/
./configure --prefix=/usr/local/python343 # 指定python343的安装目录
make && make install
cd /data/dists
mkidr py343_dists
cd py343_dists
pip install --upgrade virtualenv
virtualenv -p /usr/local/python343/bin/python3 .
source bin/activate => (py343_dists) => (in python3.4.3)
python => (Python 3.4.3)
deactivate =>(out Python 3.4.3)
```

## 3.把项目文件放到正式环境
```bash
zip  –r x.zip /home/wwwroot/x  //绝对地址的文件及文件夹进行压缩.以下给出压缩相对路径目录
scp x.zip root@ip.0.0.1:/data/dists/py343_dists
```

## 4.利用pip安装python项目依赖
```bash
pip install -r requirements.txt
when install psycopg2 //和postgresql数据库连接有关的依赖包
Error => 
yum install postgresql-devel (centOS)
apt-get install python-psycopg2 (ubuntu 16)
```

## 5.测试项目跑起来
```bash
cd project_dir
pip install Django==1.8.5 #如果没有安装django
pip install django-environ==0.4.0
python manage.py =>ok
python manage.py runserver =>ok
python manage.py migrate # 迁移数据库表
```

## 6.数据库
### 6.1安装数据库(centOS 7)
https://hmj1008.github.io/2017/04/13/linux%E4%B8%8Bdjango%E5%B8%B8%E7%94%A8%E9%83%A8%E7%BD%B2%E9%85%8D%E7%BD%AE/
### 6.2给项目配置数据库
```bash
sudo -u postgres psql
create user [hmj] with password '[hmj.1234]';
create database [hmjdb] owner [hmj];
grant all privileges on database [hmjdb] to [hmj];
cd project_dir
vim project_name.env 
```

## 7.uwsgi运行项目
```bash
cd project_dir
pip install uwsgi
uwsgi --ini uwsgi.ini 
```

## 8.前台重启uwsgi项目
```bash
ps -aux | grep uwsgi
kill -9 [pid]
uwsgi --ini uwsgi.ini
```

## 9.添加uwsgi为系统服务，后台运行
```bash
vim /usr/lib/systemd/system/uwsgi_liang.service
[Unit]
Description=uWSGI Emperor
After=syslog.target

[Service]
ExecStart=/data/dists/py343_dists/bin/uwsgi --emperor /data/dists/py343_dists/liang/uwsgi.ini
Restart=always
KillSignal=SIGQUIT
Type=notify
StandardError=syslog
NotifyAccess=all

[Install]
WantedBy=multi-user.target
systemctl start uwsgi_liang.service #启动服务
systemctl stop uwsgi_liang.service #关闭服务
```

## 10.安装nginx (ubuntu 16) 以及配置uwsgi
```bash
apt-get update
apt-get install nginx
service nginx status //在不在运行
```

project_name.nginx.conf
```bash
upstream project_sock {
    server unix:///var/uwsgi_sockets/project_name.sock;
}

server {
    listen       443;
    listen       [::]:443;
    server_name  project_name.project_name.com;

    ssl on;
    ssl_certificate   cert/project_dir/project_name.pem;
    ssl_certificate_key  cert/project_dir/project_name.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
  
    location / {
        uwsgi_pass project_sock;
        include /etc/nginx/uwsgi_params;
        fastcgi_read_timeout 120s;
        fastcgi_send_timeout 120s;
        uwsgi_read_timeout 60s;
    }
    
    location /static {
        root /data/dists/static_files/project_name;
    }

    location /media {
        root /data/dists/media_files/project_name;
    }
    
    location /images {
        root /home/www/nginx;
    }
    
    charset utf-8-r;
    access_log  /var/log/nginx/log/project_name.access.log combined;
    error_log /var/log/nginx/log/project_name.error.log warn;
}
```

uwsgi_params

```bash
uwsgi_param  QUERY_STRING       $query_string;
uwsgi_param  REQUEST_METHOD     $request_method;
uwsgi_param  CONTENT_TYPE       $content_type;
uwsgi_param  CONTENT_LENGTH     $content_length;

uwsgi_param  REQUEST_URI        $request_uri;
uwsgi_param  PATH_INFO          $document_uri;
uwsgi_param  DOCUMENT_ROOT      $document_root;
uwsgi_param  SERVER_PROTOCOL    $server_protocol;
uwsgi_param  HTTPS              $https if_not_empty;

uwsgi_param  REMOTE_ADDR        $remote_addr;
uwsgi_param  REMOTE_PORT        $remote_port;
uwsgi_param  SERVER_PORT        $server_port;
uwsgi_param  SERVER_NAME        $server_name;
```

```bash
// 检查项目ngix的配置的配置文件是否ok
service nginx -t / configtest => ok => service nginx restart
// 项目静态资源处理
cd py_dists_env
source bin/activate
cd project_name
python manage.py collectstatic
systemctl stop uwsgi_liang.service #启动服务
systemctl start uwsgi_liang.service #关闭服务
```

## 11.redis
```bash
sudo apt-get install -y tcl # for test
wget http://download.redis.io/redis-stable.tar.gz
tar xvzf redis-stable.tar.gz
cd redis-stable
make
sudo make install
redis-cli

https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-redis-on-ubuntu-16-04
```




参考：
https://www.zybuluo.com/lonelinsky/note/413507
https://hmj1008.github.io/2017FF/04/13/linux下django常用部署配置/
http://www.ruanyifeng.com/blog/2014/03/server_setup.html  #Linux服务器的初步配置流程
http://python.jobbole.com/81953/ 基于Django与Celery实现异步队列任务 Redis 
https://lihz1990.gitbooks.io/transoflptg/content/  linux 性能