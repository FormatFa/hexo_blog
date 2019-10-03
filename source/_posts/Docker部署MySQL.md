---
title: Docker部署MySQL
tags:
  - Docker
categories:
- Docker
date: 2019-09-21 16:50:38
---

镜像下载:
docker search mysql,pull第一个官方的就行

支持的tag

- 8.0.17, 8.0, 8, latest
- 5.7.27, 5.7, 5
- 5.6.45, 5.6

翻译自Docker MySQL官方镜像文档

原文链接:https://github.com/docker-library/docs/tree/master/mysql

其他的官方镜像的文档都可以在https://github.com/docker-library/docs/里找到

### 启动`mysql`服务

启动MySQL很简单

```shell
用法:
docker run --name some-mysql   -e 	MYSQL_ROOT_PASSWORD=root 用户密码 -d mysql:tag 
--name 指定容器的名字
-e 指定容器的环境变量
-d 设置容器在后台运行并输出容器id
tag 指定想要的mysql版本,不加默认使用最新

创建并运行一个叫mysqlserver的mysql容器
docker run --name  mysqlserver  -e MYSQL_ROOT_PASSWORD=123456  -d mysql

ps 命令看到mysqlserver正在运行

```

<!-- more -->

### 从MySQL命令行客户端里连接mysql

这个镜像不仅可以做上面那个mysql服务端，也可以用来做客户端来连接其他容器里的mysql，或者其他地方的mysql

上面的启动的容器是mysql服务端,下面的命令可以启动一个作为mysql命令行客户端的容器，并且连接到上面启动的那个容器里的mysql



~~~shell
1.  docker  run  -it  --rm mysql mysql -h容器的ip地址 -uexample-user -p
- 查看mysqlserver容器的ip地址
bigdata@ljh-X441UVK:~$ docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'  mysqlserver
172.17.0.2
通过指定ip连接(mysql命令的-h参数)
 docker  run  -it  --rm mysql mysql -h172.17.0.2  -uroot  -p

2. 通过指定 network连接
可以直接通过容器名字连接，不用输入ip地址
- 创建一个network 网络
```shell
docker network create -d bridge mysqlnet
bigdata@ljh-X441UVK:~$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
71dfa846b9c6        bridge              bridge              local
9fef614764d3        host                host                local
87bc32c1bc3b        mysqlnet            bridge              local
6d83addc4da3        none                null                local
```
- 将mysqlserver添加到mysqlnet网络里
```shell
docker network connect mysqlnet mysqlserver
```
- mysql客户端命令行 启动时指定mysqlnet网络
启动的容器连接到网络mysqlnet后
运行的命令是 mysql -hmysqlserver  -uroot  -p
mysql -h 指定为mysqlserver这个容器名字，因为mysqlserver在这个网络里,不用写ip地址(可能在同一个网络里有host映射)
docker  run  -it --network mysqlnet --rm mysql mysql -hmysqlserver  -uroot  -p

3 . 通过--link参数 指定来连接

docker run -it  --link mysqlserver  --rm mysql  mysql -hmysqlserver -uroot -p 
~~~

连接其他的mysql,修改-h对应的即可





关于MySQL命令行的更多信息请查看:[MySQL documentation][https://dev.mysql.com/doc/en/mysql.html]

### 访问容器shell和浏览MySQL日志

`docker exec`命令可以在Docker容器里运行一个命令.下面这个命令在你的`mysql`容器里给启动一个交互式的(-it)bash shell

`docker exec -it some-mysql bash`

可以通过Docker的容器日志查看日志

`docker logs some-mysql`

### 使用自定义的MySQL配置文件

默认的配置文件在`/etc/mysql/my.cnf`,`!includedir` 里可能有额外的目录,比如`/etc/mysql/conf.d`或者`/etc/mysql/mysql.conf.d`.请在`mysql`镜像里检查相关文件

如果`/my/custom/config-file.cnf`是自定义的配置文件的目录和文件名,可以这样启动`mysql`容器(注意这个命令里只使用了自定义配置文件的目录的路径)

`docker run -name some-mysql -v /my/custom:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag`

这个命令会启动一个新的`mysql`容器,使用`/etc/mysql/my.cnf`和`/etc/mysql/conf.d/config-file.cnf`这两个配置文件的合并结果,后面那个的配置文件的配置优先

### 不用`cnf`文件来配置配置

很多配置选项可以作为标记通过`mysqld`传递.这样可以很灵活的不用`cnf`文件来自定义容器.比如,你只想改变默认的编码和Collation字符集 成UTF-8(utf8mb4)，只需运行下面的

`docker run -name some-mysql -e  MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag --character-set-server=utf8m4 --collation-server=utf8m4_unicode_ci`

如果想查看全部可以用的选项，只需要运行:

`docker run -it --rm mysql:tag --verbose --help`



### 环境变量

当你启动`mysql`镜像,你可以在`docker run`命令行里传递一个或者更多环境变量来调整MySQL的配置.注意如果启动的容器有一个已经包含数据库的数据目录,下面的变量将不会有效果:已存在的数据库不会有变化.

查看这个连接https://dev.mysql.com/doc/refman/5.7/en/environment-variables.html 查看MySQL本身支持的环境变量的文档(特殊的变量像`MSQL_HOST`,和这个镜像使用的时候会出现问题)

`MYSQL_ROOT_PASSWORD`

这个变量必须存在,指定的密码将会设置为MySQL`root`用户的密码.在上面的例子里，他是设置成`my-secret-pw`.

`MYSQL_DATABASE`

这个变量是可选的,如果指定了，在镜像启动时会用这个的值创建一个数据库.如果也指定了 user/password (下面),这个用户会被授予这个数据库的超级权限

`MYSQL_USER`,`MYSQL_PASSWORD`

这些变量是可选的，结合在一起使用来创建一个新用户和设置用户的密码.这个用户会被授予上面通

注意这里不需要用这个机制来创建root用户，root用户会用`MYSQL_ROOT_PASSWORD`变量指定的密码来创建.

`MYSQL_ALLOW_EMPTY_PASSWORD`

这是个可选的变量.设置`yes` 就允许root用户用一个空白的密码来启动容器.注意:不推荐设置成`yes`除非你知道你在干什么,因为这个会让你的MySQL实例完全没有保护,任何人都能获得完整的超级用户权限

`MYSQL_RANDOM_ROOT_PASSWORD`
s
这是一个可选变量.设置成`yes`会随机产生一个初始化密码给root用户(使用pwgen).这个生成的密码会输出到标准输出里.

`MYSQL_ONETIME_PASSWORD`

设置root 用户初始化完后就过期,第一次登录时强制修改密码.注意:这个功能只支持MySQL 5.6+.在MySQL 5.5使用会在初始化时抛出一个错误.

------

后面的未认真翻译

### Docker 隐私

作为通过环境变量来传递敏感信息的代替,`_FILE`可能是拼接到之前列出的环境变量里,初始化脚本从一个容器的文件里加载这些变量的值,这个可以用从保存在`/run/secrets/<secret_name>`文件的Docker secrets 加载密码

比如

`docker run -name some-mysql -e MYSQL_ROOT_PASSWORD_FILE=/run/secrets/mysql-root -d mysql:tag`

当前只支持`MYSQL_ROOT_PASSWORD`,`MYSQL_ROOT_HOST`,`MYSQL_DATABASE`,`MYSQL_USER`和`MYSQL_PASSWORD`

### 初始化一个新鲜的实例

当容器第一次启动时,将会创建一个指定名字的数据库和用提供的配置变量来初始化.此外,还会执行在`/docker-entrypoint-initdb.d`里的`.sh`,`.sql`和`.sql.gz`文件.文件会按照字母排序执行.你可以通过[mounting a SQL dump into that directory](https://docs.docker.com/engine/tutorials/dockervolumes/#mount-a-host-file-as-a-data-volume)和提供一个[custom images](https://docs.docker.com/reference/builder/)带有贡献的数据.SQL 文件会默认导入通过`MYSQL_DATABASE`变量指定的数据库.

### 注意事项

### 在哪里保存数据

重要的提示:这里有几个方式来保存在Docker 容器上运行的应用使用的数据,我们鼓励`mysql`镜像的用户熟悉下面可用的选项,包括:

- 让Docker 管理数据库数据的保存[通过用他内部的volume management来写入数据库文件到宿主机](https://docs.docker.com/engine/tutorials/dockervolumes/#adding-a-data-volume),这个是默认的，并且对于用户来说很容易和公平透明.缺点是这些文件很难被直接运行在宿主机上的工具和应用定位.i.e outside container
- 创建一个数据目录在宿主机上(容器外)，[在容器里挂载这个目录可见](https://docs.docker.com/engine/tutorials/dockervolumes/#mount-a-host-directory-as-a-data-volume),这样把数据库文件放在宿主机上知道的位置,并且容器宿主机上的工具和应用访问这些文件.缺点是用户需要确定目录存在，和比如权限和其他安全机制配置正确

Docker 文件是个好起点来理解不同的存储选项和变化,这里有很多blog和论坛帖子讨论和给建议 。简单展示下上面后面选项的基本流程

1. 在宿主系统的合适volume创建一个目录,比如`/my/own/datadir`.

2. 启动`mysql`容器
   docker run --name some-mysql -v /my/own/datadir:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

   命令的`-v /my/own/datadir:/var/lib/mysql`部分,从宿主下挂在`/my/own/datadir`作为容器里的`/var/lib/mysql`,那里是MySQL默认的数据文件的地方



### 直到MySQL初始化完成之前没有连接

当容器启动时如果没有数据库初始化,将会创建一个默认的数据库.这个是预期的行为,这意味着它将不会接受传入的连接直到初始化完成.这个会在使用自动化工具时有问题,比如`docker-compose`,这个会同时启动几个容器

如果你尝试连接MySQL的应用没有处理MySQL 停机或者等待MySQL完美启动,在服务启动前放一个连接重试是必须的.在官方镜像里这样的实现,[WordPress](https://github.com/docker-library/wordpress/blob/1b48b4bccd7adb0f7ea1431c7b470a40e186f3da/docker-entrypoint.sh#L195-L235)或者[Bonita](https://github.com/docker-library/docs/blob/9660a0cccb87d8db842f33bc0578d769caaf3ba9/bonita/stack.yml#L28-L44)

### 对于已存在的数据库的用法

如果你启动一个有数据目录的`mysql`容器,而且存在了一个数据库(specifically,一个`mysql`子目录),应该从命令行里提交一个`$MYSQL_ROOT_PASSWORD`变量;他无论如何都会被忽略,并且已存在的数据库不会改变

### 以任意的用户运行

如果你知道你目录的权限已经设置合理(比如对于一个已存在的数据库运行,像上面说的)或者你需要指定mysqld运行的UID/GID.可以用`--user`来调用镜像设置任何值来完成想要的权限/配置

```
mkdir data
ls -lnd data
 docker run -v "$PWD/data":/var/lib/mysql --user 1000:1000 --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

```

### 创建数据库导出

大多数正常的工具都能用,尽管他们的用法有一点令人费解 在一些场合来确保他们有权限访问`mysqld`服务.一个简单的方法来确保就是使用`docker exec`和从相同的容器里运行工具,像下面这样:

`docker exec some-mysql sh -c 'exec mysqldump --all-databases -uroot -p"$MYSQL_ROOT_PASSWORD"' > /some/path/on/your/host/all-databases.sql`

### 从导出文件里恢复数据

对于恢复数据,你可以用`-i`选项的`docker exec`命令,像下面这样:

`$ docker exec -i some-mysql sh -c 'exec mysql -uroot -p"$MYSQL_ROOT_PASSWORD"' < /some/path/on/your/host/all-databases.sql`