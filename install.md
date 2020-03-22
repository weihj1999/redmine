# 2. 安装Redmine


## 2.1 Docker镜像

目前Docker hub提供两种镜像
- redmine官方镜像<br>
- bitnami镜像<br>

1. redmine官方镜像<br>
[官方网址](https://hub.docker.com/_/redmine)

我们首先采用官方镜像进行试验。

2. bitnami镜像<br>
[官方网址](https://hub.docker.com/r/bitnami/redmine/)

## 2.2 安装运行Redmine容器

1. 运行一个数据库实例，比如mysql

```bash
$ docker run -d --name some-mysql --network some-network -e MYSQL_USER=redmine -e MYSQL_PASSWORD=secret -e MYSQL_DATABASE=redmine -e MYSQL_RANDOM_ROOT_PASSWORD=1 mysql:5.7
```

2. 运行Redmine

```bash
$ docker run -d --name some-redmine --network some-network -e REDMINE_DB_POSTGRES=some-postgres -e REDMINE_DB_USERNAME=redmine -e REDMINE_DB_PASSWORD=secret redmine
```

>注意:
上述实例，并不保证持久性数据保持。

## 2.3 配置选项

## 2.3.1 数据保留
在docker容器中运行的应用有很多种方式来存储数据，redmine推荐使用以下两种方式：
- 让Docker通过使用其自身的内部卷管理将文件写入主机系统上的磁盘来管理文件存储。 这是默认设置，对用户而言是简单且相当透明的。 缺点是，对于直接在主机系统（即外部容器）上运行的工具和应用程序，文件可能很难定位。
- 在主机系统上（容器外部）创建一个数据目录，并将其安装到从容器内部可见的目录中。 这会将数据库文件放置在主机系统上的已知位置，并使主机系统上的工具和应用程序易于访问文件。 缺点是用户需要确保该目录存在，比如正确设置主机系统上的目录权限和其他安全机制。

Docker文档可以帮助用户了解不同存储选项和其他可选项，并且有很多博客和论坛文章讨论了这一领域的问题并提供了建议。 我们将在此处描述后一种方法的基本过程：

1. 在主机系统上创建一个数据目录，比如~/redminedata
2. 按照下面的方式来启动redmine容器：

```bash
$ docker run -d --name some-redmine -v ~/redminedata:/usr/src/redmine/files --link some-postgres:postgres redmine
```

命令中的-v \~/redminedata:/usr/src/redmine/files会把目录~/redminedata以/usr/src/redmine/files的形式挂在到容器内部，用于Redmine管理上传的文件。

下面主要讨论一下，redmine容器运行的时候需要重点关注的配置选项，

### 2.3.2 端口映射

如果希望能够在没有容器IP的情况下从主机访问实例，则可以使用标准端口映射。 只需将-p 3000:3000添加到docker run参数中，然后在浏览器中访问http://localhost:3000或http://host-ip:3000。

所以操作指导，我们这里仅进行3000端口的映射即可。

### 2.3.3 环境变量

1. REDMINE_DB_MYSQL or REDMINE_DB_POSTGRES

这两个变量是互斥的，用于设置数据库主机的IP地址或者主机名。如果都设置，则相当于没有设置，都没有设置，则使用默认设置SQLite

2. REDMINE_DB_PORT

设置数据库的访问端口，如果没有设置，则MySQL默认使用3306， PostGreSQL默认使用5432， SQLite则默认使用空。

3. REDMINE_DB_USERNAME

设置redmine访问数据库的用户，如果不指定，MySQL默认使用*root*， PostgreSQL默认使用*postgres*, SQLite则默认使用*redmine*

4. REDMINE_DB_PASSWORD

设置redmine访问数据库使用的密码，这里没有默认值。

5. REDMINE_DB_DATABASE

redmine需要访问的数据库，如果不指定，MySQL默认使用*redmine*， PostgreSQL默认使用*REDMINE_DB_USERNAME*的值, SQLite则默认使用*sqlite/redmine.db*

6. 配置所有数据库支持utf-8

创建redmysql.cnf，内容如下：
```
[mysqld]
character-set-server = utf8
collation-server = utf8_unicode_ci
skip-character-set-client-handshake
```

参数
```
...
-v redmysql.cnf:/etc/mysql/conf.d/custom.cnf
...
```
7. 其他参数根据情况使用，这里不再意义列举，具体参考官方指导手册。


## 2.4 安装试验

1. 运行的命令

- 运行MySQL

```bash
$ docker run -d --name redminedb \
--restart always --net shared_nw --ip 10.1.1.109 \
-p 3309:3306 \
-e MYSQL_USER=redmine \
-e MYSQL_PASSWORD=Secret109 \
-e MYSQL_DATABASE=redmine \
-e MYSQL_RANDOM_ROOT_PASSWORD=Q1w2e3r4 \
-v ~/redminedbdata:/var/lib/mysql \
mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

>注意:<br>
这里的--character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci，如果不加上，redmine里使用中文会有问题。

- 运行Redmine

```bash
$ docker run -d --name redmine \
-v ~/redminedata:/usr/src/redmine/files \
-p 3010:3000 \
--restart always --net shared_nw --ip 10.1.1.110 \
-e REDMINE_DB_MYSQL=redminedb \
-e REDMINE_DB_PASSWORD=Secret109 \
-e REDMINE_DB_USERNAME=redmine \
redmine
```

2. 服务初始化

默认的登录用户和密码： admin/admin
