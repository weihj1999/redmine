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

### 2.3.2

## 2.4 安装试验

1. 运行的命令

```bash

```

2. 服务初始化
