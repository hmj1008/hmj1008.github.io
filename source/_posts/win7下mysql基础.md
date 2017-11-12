---
title: win7下mysql基础
date: 2017-04-13 07:16:03
toc: true
tags:
    - sql
    - 代码
---
老早前从网上扒来的一些浅显资料。充个数，作个字典，也许日后会翻翻。
<!--more-->
## 1，启动停止(win7)
```html
cmd
    net start mysql
    net stop mysql
```

## 2，MySQL的登录，退出
```html
登录：
    mysql -uroot -p -P3306 -h127.0.0.1
    -u:用户名;     -p:密码;
    -P:端口号;     -h:服务器名称;
    -D:打开指定数据库;    --delimiter:指定分割符
    -V:输入版本信息并退出; --prompt:设置提示符

退出:
    exit;
    quit;
    \q;
```
## 3，修改MySQL提示符
```html
1.连接客户端时通过参数指定
    shell>mysql -uroot -proot --prompt \h
2.连上客户端后，通过prompt修改
    mysql>prompt 提示符号
3.注
   \h=>localhost
   \D=>完整日期
   \d=>打开的数据库名称
   \u=>当前用户
```

## 4，MySQL常用命令
```sql
常用命令：
    SELECT VERSION();//显示当前服务器版本
    SELECT NOW();//显示当前日期时间
    SELECT USER();//显示当前用户
语句规范：
    关键字与函数名称全部用大写;
    数据库名称、表名称、字段名称用小写;
    SQL语句必须以分号结尾;
```
## 5，创建数据库,设置编码，修改编码，删除
```sql
FullSql:
    CREATE {DATABASE | SCHEMA} [IF NOT EXISTS] db_name [DEFAULT] CHARACTER SET [=] charset_name
    /*{==>内容必须*/
    /*|==>内容可选*/
    /*[==>内容可要可不要*/ 
e.g: 
    CREATE DATABASE t1;
    CREATE DATABASE t2 IF NOT EXISTS t2 CHARSET gbk;
    /*查看当前服务器下的数据库列表*/
    SHOW DATABASES;
    /*查看错误信息*/
    SHOW WARNINGS;
    /*展示创建该数据库时的信息，可以查看编码信息*/
    SHOW CREATE DATABASE t1;
    /*修改编码*/
    ALTER DATABASE t2 CHARSET SET = utf-8;
    /*删除数据库*/
    DROP DATABASE t1;
```
## 6，数据类型
```sql
整数：
    tinyint	    允许从 0 到 255 的所有数字。	           1 字节
    smallint	允许从 -32,768 到 32,767。	           2 字节
    int	        允许从 -2,147,483,648 到 2,147,483,647  4 字节
    bigint	    允许介于                                8 字节
浮点数：
    FLOAT(size,d)	
    带有浮动小数点的小数字。在括号中规定最大位数。在 d 参数中规定小数点右侧的最大位数。
    DOUBLE(size,d)	
    带有浮动小数点的大数字。在括号中规定最大位数。在 d 参数中规定小数点右侧的最大位数。
日期：
    DATE()
        //日期。格式：YYYY-MM-DD
        注释：支持的范围是从 '1000-01-01' 到 '9999-12-31'
    DATETIME()
        *日期和时间的组合。格式：YYYY-MM-DD HH:MM:SS
        注释：支持的范围是从 '1000-01-01 00:00:00' 到 '9999-12-31 23:59:59'
    TIMESTAMP()	
        *时间戳。TIMESTAMP 值使用 Unix 纪元('1970-01-01 00:00:00' UTC) 至今的描述来存储。格式：YYYY-MM-DD HH:MM:SS
        注释：支持的范围是从 '1970-01-01 00:00:01' UTC 到 '2038-01-09 03:14:07' UTC

字符型：
    CHAR(size)	
        保存固定长度的字符串（可包含字母、数字以及特殊字符）。最多 255 个字符。
    VARCHAR(size)	
        保存可变长度的字符串（可包含字母、数字以及特殊字符）。最大长度255 
        注释：如果值的长度大于 255，则被转换为 TEXT 类型。
    TEXT	
        存放最大长度为 65,535 个字符的字符串。
    BLOB	
        用于 BLOBs (Binary Large OBjects)。存放最多 65,535 字节的数据。
    MEDIUMTEXT	
        存放最大长度为 16,777,215 个字符的字符串。
    MEDIUMBLOB	
        用于 BLOBs (Binary Large OBjects)。存放最多 16,777,215 字节的数据。
    LONGTEXT	
        存放最大长度为 4,294,967,295 个字符的字符串。
    LONGBLOB	
        用于 BLOBs (Binary Large OBjects)。存放最多 4,294,967,295 字节的数据。
    ENUM(x,y,z,etc.)	
        允许你输入可能值的列表。可以在 ENUM 列表中列出最大 65535 个值。如果列表中不存在插入的值，则插入空值。
    注释：这些值是按照你输入的顺序存储的。
    可以按照此格式输入可能的值：ENUM('X','Y','Z')
    SET	与 ENUM 类似，SET 最多只能包含 64 个列表项，不过 SET 可存储一个以上的值。
```
## 7，创建/修改数据表,插入数据,简单select
```sql
CREATE:
    CREATE TABLE [IF NOT EXITS] table_name(
        column_name datatype,
    )

ALTER：
/*MEDIUMTEXT 存放最大长度为 16,777,215 个字符的字符串。*/
ALTER TABLE goods_detail MODIFY COLUMN detail_info MEDIUMTEXT;

e.g:
    CREATE TABLE tb1(
        id INT,
        user_name VARCHAR(20),
        age TINYINT UNSIGNED,
        salary FLOAT(8,3) UNSIGNED
        ); 
        
    CREATE TABLE tb2(
        id INT AUTO_INCREMENT PRIMARY KEY, /*主键，自动递增+1*/
        user_name VARCHAR(20) NOT NULL,    /*设置不允许为空*/
        age TINYINT UNSIGNED,
        class_name VARCHAR(20) UNIQUE KEY,/*唯一约束*/
        birth_year INT UNIQUE KEY,        /*唯一约束*/
        sex ENUM(`1`,`2`,`3`) DEFAULT `3`, /*默认3*/
        salary FLOAT(8,3) UNSIGNED/*整个浮点数最大8位，小数点后面最多3位，无符号*/
        );   
          
    SHOW TABLES [FROM db_name] /*查看数据表列表*/
    SHOW COLUMNS FROM tb_name /*查看数据表结构*/
Insert:
    INSERT INTO tb1 VALUES(1,`TOM`,20,4500.78);
    INSERT INTO tb1 (id,user_name,salary) VALUES (2,`John`,4000.77);
Select:
    SELECT * FROM tb1;
    
```
## 8，约束
```sql
约束类型：
    NOT NULL
    PRIMARY KEY
    UNIQUE KEY
    DEFAULT
    FOR外键约束
        外键约束的要求
        1,父表    
```