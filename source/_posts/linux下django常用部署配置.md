---
title: linux下django常用部署配置
date: 2017-04-13 08:38:35
toc: true
tags:
    - django
    - linux
    - 代码
---
> 
时间回到2016年的八月，鬼知道我经历了什么。阴差阳错从android,spring的Java世界来到了linux下的python开发，一直到现在。在这里把一些开发常用的命令以及一些配置套路记录一下。有比较才有收获，当然这些收货不是在这里三言两语可以表达的。有人说：人生苦短我用python。也有人说：Java是世界最好的语言。我想说：语言嘛只是工具，&( ^___^ )&
<!--more-->

## Linux下的django开发环境搭建
### Install django through pip in a Virtualenv
由于一般linx系统自带默认python版本是pyhon2.7，开发大家普遍采用3.X,所以需要用Virtualenv工具虚拟出一个python3.X的环境，简单方便干脆。django的开发完成后，部署在生产环境涉及到postgresql,uwsgi,nginx,supervisord等等软件的安装配置，协同工作。
```bash
OS: ubuntu-16

sudo apt-get update

sudo apt-get install python3-pip

sudo pip3 install virtualenv

$ virtualenv -p /usr/bin/python3 py3env

mkdir ~/newproject

cd ~/newproject

virtualenv py3env

source py3env/bin/activate <-->deactivate

//In your new environment, you can use pip to install Django. Regardless of 

//whether you are using version 2 or 3 of Python, it should be called just pip 

//when you are in your virtual environment. Also note that youdo not need 

//to use sudo since you are installing locally:

pip install django

django-admin --version
```

### Creating a Sample Project
```bash
// create a django project
django-admin startproject projectname

//change to the projects
cd projectname

// config database init data
python manage.py migrate

// create an administrative user by typing:
python manage.py createsuperuser
      
// run
python manage.py runserver

// 创建应用   
python manage.py startapp app_name


// 添加应用 /projectName/projectName/settings.py

INSTALLED_APPS = [
'polls.apps.PollsConfig',
'django.contrib.admin',
...
]

// after model python code ,migrate database
python manage.py makemigrations
python manage.py migrate
```

## Deploy Django Project in production(ubuntu-16/centos-7)
### python环境以及项目依赖   
```python
// Install django through pip in a Virtualenv

// 生成项目所依赖的第三方模块名称以及版本信息写入到requirements.txt文件
pip freeze > requirements.txt

// 安装requirements.txt中所有的依赖
pip install -r requirements.txt
```

### 安装数据库 postgresql(ubuntu-16/centos-7)
```bash
ubuntu-16-04 (apt-get):
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib
sudo apt-get install python-psycopg2
sudo apt-get install libpq-dev

centos-7 (yum):
// 1.安装 yum 源（地址从 http://yum.postgresql.org/repopackages.php 获取）
sudo yum install http://yum.postgresql.org/9.5/redhat/rhel-7-x86_64/pgdg-centos95-9.5-2.noarch.rpm

// 2.安装 postgresql95-server 和 postgresql95-contrib
sudo yum install postgresql95-server postgresql95-contrib

// 3.安装后，可执行文件在 /usr/pgsql-9.5/bin/， 数据和配置文件在 /var/lib/pgsql/9.5/data/

// 4.初始化数据:
sudo /usr/pgsql-9.5/bin/postgresql95-setup initdb

// 5.默认不支持密码认证，修改 pg_hab.conf 将 ident 替换为 md5 （可选）
sudo vim /var/lib/pgsql/9.5/data/pg_hba.conf

// 6.微调其他配置项（可选）
sudo vim /var/lib/pgsql/9.5/data/postgresql.conf

// 7.启动服务：
sudo systemctl start postgresql-9.5.service

// 8.开机自动启动:
sudo systemctl enable postgresql-9.5.service
```
     
### 创建数据库用户以及项目数据库
```bash
// 用postgres用户登录到数据库
sudo -u postgres psql
// 创建用户
create user [hmj] with password '[hmj.1234]'
create database [hmjdb] owner [hmj]
grant all privileges on database [hmjdb] to [hmj]

// postgresql基本命令
\h：查看SQL命令的解释，比如\h select。
\?：查看psql命令列表。
\l：列出所有数据库。
\c [database_name]：连接其他数据库。
\d：列出当前数据库的所有表格。
\d [table_name]：列出某一张表格的结构。
\du：列出所有用户。
\e：打开文本编辑器。
\conninfo：列出当前数据库和连接的信息。


# 创建新表 
CREATE TABLE user_tbl(name VARCHAR(20), signup_date DATE);

# 插入数据 
INSERT INTO user_tbl(name, signup_date) VALUES('张三', '2013-12-22');

# 选择记录 
SELECT * FROM user_tbl;

# 更新数据 
UPDATE user_tbl set name = '李四' WHERE name = '张三';

# 删除记录 
DELETE FROM user_tbl WHERE name = '李四' ;

# 添加栏位 
ALTER TABLE user_tbl ADD email VARCHAR(40);

# 更新结构 
ALTER TABLE user_tbl ALTER COLUMN signup_date SET NOT NULL;

# 更名栏位 
ALTER TABLE user_tbl RENAME COLUMN signup_date TO signup;

# 删除栏位 
ALTER TABLE user_tbl DROP COLUMN email;

# 表格更名 
ALTER TABLE user_tbl RENAME TO backup_tbl;

# 删除表格 
DROP TABLE IF EXISTS backup_tbl;
```
### uwsgi配置django
```bash
apt-get install python-dev #不安装这个，下面的安装可能会失败
pip install uwsgi
```

```bash
# mysite_uwsgi.ini file
[uwsgi]

# Django-related settings
# the base directory (full path)
#chdir = /data/dists/py3_dists/PShop
chdir = /home/hy/py3env/fv/FindV

# Django's wsgi file
module = FindV.wsgi

# the virtualenv (full path) 
#home = /path/to/virtualenv
home = /home/hy/py3env

# process-related settings
uid = www-data
gid = www-data

#master
master = true
# maximum number of worker processes
processes = 1 
threads = 4  # nginx process +4

# the socket (use the full path to be safe) 
socket = /var/uwsgi_sockets/findv.sock
# with appropriate permissions - may be needed
chmod-socket = 666

# clear environment on exit
vacuum = true

# log and stats
stats = 127.0.0.1:9395
#stats = 121.40.134.238:9394
logto = /var/log/uwsgi/uwsgi_findv.log
logfile-chown = true
py-tracebacker=/var/uwsgi_sockets/findv_tbsocket
```
### nginx 配置http(s)
```bash
upstream django_findv {
    server unix:///var/uwsgi_sockets/findv.sock;
}

server {

    listen 80;
    
    # https 配置开始
    listen 443;
    ssl on;
    ssl_certificate cert/12344XXXX.pem;
    ssl_certificate_key cert/12344XXXX.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    # https 配置结束
    
    #server_name  www.mjcode.cn;
    server_name localhost;
    client_max_body_size 20m;#表示最大上传20M，需要多大设置多大。

    location /static {
        root /data/dists/static_files/FindV;
        #access_log   off;
        #expires      30d;
    }

    location /media {
        root /data/dists/static_files/FindV;
        #access_log   off;
        #expires      30d;
    }

    location / {
        uwsgi_pass django_findv;
        include /etc/nginx/uwsgi_params;
        fastcgi_read_timeout 120s;
        fastcgi_send_timeout 120s;
        uwsgi_read_timeout 60s;
    }

    location /images {
        root /home/www/nginx;
    }

    charset utf-8-r;
    access_log  /var/log/nginx/log/findv.access.log combined;
    error_log /var/log/nginx/log/finv.error.log warn;
}
```
### git命令以及最近踩过的一个坑
```bash
// 基本git命令
查看分支：git branch
创建分支：git branch <name>
切换分支：git checkout <name>
创建+切换分支：git checkout -b <name>
合并某分支到当前分支：git merge <name>
删除分支：git branch -d <name>

// 最近踩过的一个坑
// 错误姿势：
.gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。那么解决方法就是先把本地缓存删除（改变成未track状态），然后再提交：
git rm -r --cached .
git add .
git commit -m 'update .gitignore'
注意：以上做法是错误的，这样会造成远程仓库的符合.gitignore描述的文件被删除！
// 正确姿势：
1,备份已跟踪但想在以后的操作总通过.gitignore忽略的文件。
2,删除文件，提交。
3,恢复第一步备份。此时生效。
```

### 安装supervisord（Centos 7）
```bash
// install supervisord
yum install python-setuptools
easy_install supervisor
echo_supervisord_conf > /etc/supervisord.conf

// vim /etc/supervisord.conf
[program:FindV]
command=/usr/local/bin/uwsgi --ini /etc/uwsgi/findv_uwsgi.ini
directory=/data/dists/py35_dists/FindV
startsecs=0
stopwaitsecs=0
autostart=true
autorestart=true

// 利用supervisord启动uwsgi拉起django项目（前提是uwsgi 和ngingx 配置好)
supervisorctl -c /etc/supervisord.conf restart FindV
```

