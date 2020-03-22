# 4. 常见问题

## 4.1 github的问题

Atom不支持自动注册远程github，需要通过命令行去添加remote repository。 比如：

git remote add origin https://github.com/weihj1999/redmine.git
git push -u origin master

然后点击右下角的github标签，会提示登录 github.atom.io/login, 需要注册一个atom的账号。

获取一个token后可以正常登录。

它会自动识别你的githug账号，授权登录即可。

然后在git标签进行stage，commit和推送即可了。

## 4.2 -link关联数据库

这个功能已经不推荐使用，我们使用环境变量来做容器端口和IP信息的传递。

具体参考安装部分如何制定数据库。

## 4.3 配置问题

1. 配置邮件通知

默认情况下，进入配置管理，点击邮件通知标签，显示如下：

```
邮件参数尚未配置，因此邮件通知功能已被禁用。
请在config/configuration.yml中配置您的SMTP服务器信息并重新启动以使其生效。
```

[官方手册](https://www.redmine.org/projects/redmine/wiki/emailconfiguration)

其中Gmail的配置样例：

/usr/src/redmine/config/configuration.yml

```
production:
  email_delivery:
    delivery_method: :smtp
    smtp_settings:
      enable_starttls_auto: true
      address: "smtp.gmail.com"
      port: 587
      domain: "smtp.gmail.com"
      authentication: :plain
      user_name: "mymail@gmail.com"
      password: "mypass"
```

阿里企业邮箱配置：

```
email_delivery:
    delivery_method: :smtp
    smtp_settings:
      enable_starttls_auto: true
      address: smtp.qiye.aliyun.com
      port: 465
      ssl: true
      domain: smtp.qiye.aliyun.com # 'your.domain.com' for GoogleApps
      authentication: :login
      user_name: 'my@email.com'
      password: 'mypass'
```

## 4.4 不能更新中文字符

现象： 创建中文字符的项目，问题等等出现错误：
```
Mysql2::Error: Incorrect string value: '\xE7\xAE\xA1\xE7\x90\x86...' for column 'name' at row 1: INSERT INTO `roles` (`name`, `position`, `issues_visibility`) VALUES ('管理人员', 1, 'all')
```

一般来说Mysql(小于5.5.3)字符集设置为utf8,指定连接的字符集也为utf8，django中save unicode string是木有问题的。但是，当字符串中有特殊字符（如emoji表情符号，以及其他凡是转成utf8要占用4字节的字符），就会有问题，会报错Incorrect string value: ‘\xF0\x9F\x91\x93\xF0\x9F…’ for column ‘xxx’ at row 1

大家都知道Unicode是一个标准，utf8是unicode一个实现方式, 某些Unicode字符转成utf8可能4字节，而在MySQl5.5.3之前，utf8最长只有3字节


mysql5.7版本之后，参考下面的办法：

$ docker cp redminedb:/etc/mysql/conf.d/mysql.cnf .
$ vi mysql.cnf
按照下面的内容修改保存，
```
[client]                              
default-character-set = utf8mb4       

[mysql]                               
default-character-set = utf8mb4       

[mysqld]                              
character-set-client-handshake = FALSE
character-set-server = utf8mb4        
collation-server = utf8mb4_unicode_ci
init_connect='SET NAMES utf8mb4'      
```
重新启动容器
```bash
$ docker stop redminedb
$ docker start redminedb
```

检查：
```
[weihj@cicdhosts ~]$ docker exec -it redminedb bash
root@2203dca70979:/# mysql -h 10.1.1.109 -u redmine -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.29 MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show character set;
+----------+---------------------------------+---------------------+--------+
| Charset  | Description                     | Default collation   | Maxlen |
+----------+---------------------------------+---------------------+--------+
| big5     | Big5 Traditional Chinese        | big5_chinese_ci     |      2 |
| dec8     | DEC West European               | dec8_swedish_ci     |      1 |
| cp850    | DOS West European               | cp850_general_ci    |      1 |
| hp8      | HP West European                | hp8_english_ci      |      1 |
| koi8r    | KOI8-R Relcom Russian           | koi8r_general_ci    |      1 |
| latin1   | cp1252 West European            | latin1_swedish_ci   |      1 |
| latin2   | ISO 8859-2 Central European     | latin2_general_ci   |      1 |
| swe7     | 7bit Swedish                    | swe7_swedish_ci     |      1 |
| ascii    | US ASCII                        | ascii_general_ci    |      1 |
| ujis     | EUC-JP Japanese                 | ujis_japanese_ci    |      3 |
| sjis     | Shift-JIS Japanese              | sjis_japanese_ci    |      2 |
| hebrew   | ISO 8859-8 Hebrew               | hebrew_general_ci   |      1 |
| tis620   | TIS620 Thai                     | tis620_thai_ci      |      1 |
| euckr    | EUC-KR Korean                   | euckr_korean_ci     |      2 |
| koi8u    | KOI8-U Ukrainian                | koi8u_general_ci    |      1 |
| gb2312   | GB2312 Simplified Chinese       | gb2312_chinese_ci   |      2 |
| greek    | ISO 8859-7 Greek                | greek_general_ci    |      1 |
| cp1250   | Windows Central European        | cp1250_general_ci   |      1 |
| gbk      | GBK Simplified Chinese          | gbk_chinese_ci      |      2 |
| latin5   | ISO 8859-9 Turkish              | latin5_turkish_ci   |      1 |
| armscii8 | ARMSCII-8 Armenian              | armscii8_general_ci |      1 |
| utf8     | UTF-8 Unicode                   | utf8_general_ci     |      3 |
| ucs2     | UCS-2 Unicode                   | ucs2_general_ci     |      2 |
| cp866    | DOS Russian                     | cp866_general_ci    |      1 |
| keybcs2  | DOS Kamenicky Czech-Slovak      | keybcs2_general_ci  |      1 |
| macce    | Mac Central European            | macce_general_ci    |      1 |
| macroman | Mac West European               | macroman_general_ci |      1 |
| cp852    | DOS Central European            | cp852_general_ci    |      1 |
| latin7   | ISO 8859-13 Baltic              | latin7_general_ci   |      1 |
| utf8mb4  | UTF-8 Unicode                   | utf8mb4_general_ci  |      4 |
| cp1251   | Windows Cyrillic                | cp1251_general_ci   |      1 |
| utf16    | UTF-16 Unicode                  | utf16_general_ci    |      4 |
| utf16le  | UTF-16LE Unicode                | utf16le_general_ci  |      4 |
| cp1256   | Windows Arabic                  | cp1256_general_ci   |      1 |
| cp1257   | Windows Baltic                  | cp1257_general_ci   |      1 |
| utf32    | UTF-32 Unicode                  | utf32_general_ci    |      4 |
| binary   | Binary pseudo charset           | binary              |      1 |
| geostd8  | GEOSTD8 Georgian                | geostd8_general_ci  |      1 |
| cp932    | SJIS for Windows Japanese       | cp932_japanese_ci   |      2 |
| eucjpms  | UJIS for Windows Japanese       | eucjpms_japanese_ci |      3 |
| gb18030  | China National Standard GB18030 | gb18030_chinese_ci  |      4 |
+----------+---------------------------------+---------------------+--------+
41 rows in set (0.00 sec)

mysql> show variables like 'char%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8mb4                    |
| character_set_connection | utf8mb4                    |
| character_set_database   | utf8mb4                    |
| character_set_filesystem | binary                     |
| character_set_results    | utf8mb4                    |
| character_set_server     | utf8mb4                    |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
```

中文字符项目仍然还有问题，不能正确保存

下面的方法，登录mysql
```
mysql> use redmine;
mysql> alter table projects CONVERT TO CHARACTER SET utf8;
Query OK, 1 row affected (0.36 sec)
Records: 1  Duplicates: 0  Warnings: 0
mysql>
```
验证通过；
