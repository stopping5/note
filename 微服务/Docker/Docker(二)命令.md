# 命令

##  一、容器生命周期管理

###  1、run

语法：

```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

OPTIONS:

```
-a stdin: 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；

-d: 后台运行容器，并返回容器ID；

-i: 以交互模式运行容器，通常与 -t 同时使用；

-P: 随机端口映射，容器内部端口随机映射到主机的端口

-p: 指定端口映射，格式为：主机(宿主)端口:容器端口

-t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；

--name="nginx-lb": 为容器指定一个名称；

--dns 8.8.8.8: 指定容器使用的DNS服务器，默认和宿主一致；

--dns-search example.com: 指定容器DNS搜索域名，默认和宿主一致；

-h "mars": 指定容器的hostname；

-e username="ritchie": 设置环境变量；

--env-file=[]: 从指定文件读入环境变量；

--cpuset="0-2" or --cpuset="0,1,2": 绑定容器到指定CPU运行；

-m :设置容器使用内存最大值；

--net="bridge": 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；

--link=[]: 添加链接到另一个容器；

--expose=[]: 开放一个端口或一组端口；

--volume , -v: 绑定一个卷
```



### 2、start/stop/restart

**docker start** :启动一个或多个已经被停止的容器

**docker stop** :停止一个运行中的容器

**docker restart** :重启容器

### 3、kill 

**docker kill** :杀掉一个运行中的容器。

OPTIONS说明：

```
-s :向容器发送一个信号
```

### 4、rm

**docker rm ：**删除一个或多个容器。

```
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

OPTIONS说明：

```
-f :通过 SIGKILL 信号强制删除一个运行中的容器。
-l :移除容器间的网络连接，而非容器本身。
-v :删除与容器关联的卷。
```

### 实例

强制删除容器 db01、db02：

```
docker rm -f db01 db02
```

移除容器 nginx01 对容器 db01 的连接，连接名 db：

```
docker rm -l db 
```

删除容器 nginx01, 并删除容器挂载的数据卷：

```
docker rm -v nginx01
```

删除所有已经停止的容器：

```
docker rm $(docker ps -a -q)
```

### 5、pause/unpause

**docker pause** :暂停容器中所有的进程。

**docker unpause** :恢复容器中所有的进程。

### 语法

```
docker pause [OPTIONS] CONTAINER [CONTAINER...]
docker unpause [OPTIONS] CONTAINER [CONTAINER...]
```

### 实例

暂停数据库容器db01提供服务。

```
docker pause db01
```

恢复数据库容器db01提供服务。

```
docker unpause db01
```

### 6、create

**docker create ：**创建一个新的容器但不启动它

### 语法

```
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

语法同 [docker run](https://www.runoob.com/docker/docker-run-command.html)

### 实例

使用docker镜像nginx:latest创建一个容器,并将容器命名为myrunoob

```
runoob@runoob:~$ docker create  --name myrunoob  nginx:latest      
09b93464c2f75b7b69f83d56a9cfc23ceb50a48a9db7652ee4c27e3e2cb1961f
```

### 7、exec

**docker exec ：**在运行的容器中执行命令

### 语法

```
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

OPTIONS说明：

- **-d :**分离模式: 在后台运行
- **-i :**即使没有附加也保持STDIN 打开
- **-t :**分配一个伪终端

### 实例

在容器 mynginx 中以交互模式执行容器内 /root/runoob.sh 脚本:

```
runoob@runoob:~$ docker exec -it mynginx /bin/sh /root/runoob.sh
http://www.runoob.com/
```

在容器 mynginx 中开启一个交互模式的终端:

```
runoob@runoob:~$ docker exec -i -t  mynginx /bin/bash
root@b1a0703e41e7:/#
```

也可以通过 **docker ps -a** 命令查看已经在运行的容器，然后使用容器 ID 进入容器。

查看已经在运行的容器 ID：

```
# docker ps -a 
...
9df70f9a0714        openjdk             "/usercode/script.sh…" 
...
```

第一列的 9df70f9a0714 就是容器 ID。

通过 exec 命令对指定的容器执行 bash:

```
# docker exec -it 9df70f9a0714 /bin/bash
```



## 二、容器操作

### 1、ps

**docker ps :** 列出容器

### 语法

```
docker ps [OPTIONS]
```

OPTIONS说明：

- **-a :**显示所有的容器，包括未运行的。
- **-f :**根据条件过滤显示的内容。
- **--format :**指定返回值的模板文件。
- **-l :**显示最近创建的容器。
- **-n :**列出最近创建的n个容器。
- **--no-trunc :**不截断输出。
- **-q :**静默模式，只显示容器编号。
- **-s :**显示总的文件大小。

### 实例

列出所有在运行的容器信息。

```
runoob@runoob:~$ docker ps
CONTAINER ID   IMAGE          COMMAND                ...  PORTS                    NAMES
09b93464c2f7   nginx:latest   "nginx -g 'daemon off" ...  80/tcp, 443/tcp          myrunoob
96f7f14e99ab   mysql:5.6      "docker-entrypoint.sh" ...  0.0.0.0:3306->3306/tcp   mymysql
```

输出详情介绍：

**CONTAINER ID:** 容器 ID。

**IMAGE:** 使用的镜像。

**COMMAND:** 启动容器时运行的命令。

**CREATED:** 容器的创建时间。

**STATUS:** 容器状态。

状态有7种：

- created（已创建）
- restarting（重启中）
- running（运行中）
- removing（迁移中）
- paused（暂停）
- exited（停止）
- dead（死亡）

**PORTS:** 容器的端口信息和使用的连接类型（tcp\udp）。

**NAMES:** 自动分配的容器名称。

列出最近创建的5个容器信息。

```
runoob@runoob:~$ docker ps -n 5
CONTAINER ID        IMAGE               COMMAND                   CREATED           
09b93464c2f7        nginx:latest        "nginx -g 'daemon off"    2 days ago   ...     
b8573233d675        nginx:latest        "/bin/bash"               2 days ago   ...     
b1a0703e41e7        nginx:latest        "nginx -g 'daemon off"    2 days ago   ...    
f46fb1dec520        5c6e1090e771        "/bin/sh -c 'set -x \t"   2 days ago   ...   
a63b4a5597de        860c279d2fec        "bash"                    2 days ago   ...
```

列出所有创建的容器ID。

```
runoob@runoob:~$ docker ps -a -q
09b93464c2f7
b8573233d675
b1a0703e41e7
f46fb1dec520
a63b4a5597de
6a4aa42e947b
de7bb36e7968
43a432b73776
664a8ab1a585
ba52eb632bbd
...
```

### 2、inspect

**docker inspect :** 获取容器/镜像的元数据。

### 语法

```
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
```

OPTIONS说明：

- **-f :**指定返回值的模板文件。
- **-s :**显示总的文件大小。
- **--type :**为指定类型返回JSON。

### 实例

获取镜像mysql:5.6的元信息。

```
runoob@runoob:~$ docker inspect mysql:5.6
[
    {
        "Id": "sha256:2c0964ec182ae9a045f866bbc2553087f6e42bfc16074a74fb820af235f070ec",
        "RepoTags": [
            "mysql:5.6"
        ],
        "RepoDigests": [],
        "Parent": "",
        "Comment": "",
        "Created": "2016-05-24T04:01:41.168371815Z",
        "Container": "e0924bc460ff97787f34610115e9363e6363b30b8efa406e28eb495ab199ca54",
        "ContainerConfig": {
            "Hostname": "b0cf605c7757",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "3306/tcp": {}
            },
...
```

获取正在运行的容器mymysql的 IP。

```
runoob@runoob:~$ docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mymysql
172.17.0.3
```

### 3、top

**docker top :**查看容器中运行的进程信息，支持 ps 命令参数。

### 语法

```
docker top [OPTIONS] CONTAINER [ps OPTIONS]
```

容器运行时不一定有/bin/bash终端来交互执行top命令，而且容器还不一定有top命令，可以使用docker top来实现查看container中正在运行的进程。

### 实例

查看容器mymysql的进程信息。

```
runoob@runoob:~/mysql$ docker top mymysql
UID    PID    PPID    C      STIME   TTY  TIME       CMD
999    40347  40331   18     00:58   ?    00:00:02   mysqld
```

查看所有运行容器的进程信息。

```
for i in  `docker ps |grep Up|awk '{print $1}'`;do echo \ &&docker top $i; done
```

### 4、attach

**docker attach :**连接到正在运行中的容器。

### 语法

```
docker attach [OPTIONS] CONTAINER
```

要attach上去的容器必须正在运行，可以同时连接上同一个container来共享屏幕（与screen命令的attach类似）。

官方文档中说attach后可以通过CTRL-C来detach，但实际上经过我的测试，如果container当前在运行bash，CTRL-C自然是当前行的输入，没有退出；如果container当前正在前台运行进程，如输出nginx的access.log日志，CTRL-C不仅会导致退出容器，而且还stop了。这不是我们想要的，detach的意思按理应该是脱离容器终端，但容器依然运行。好在attach是可以带上--sig-proxy=false来确保CTRL-D或CTRL-C不会关闭容器。

### 实例

容器mynginx将访问日志指到标准输出，连接到容器查看访问信息。

```
runoob@runoob:~$ docker attach --sig-proxy=false mynginx
192.168.239.1 - - [10/Jul/2016:16:54:26 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.93 Safari/537.36" "-"
```

### 5、events

**docker events :** 从服务器获取实时事件

### 语法

```
docker events [OPTIONS]
```

OPTIONS说明：

- **-f ：**根据条件过滤事件；
- **--since ：**从指定的时间戳后显示所有事件;
- **--until ：**流水时间显示到指定的时间为止；

### 实例

显示docker 2016年7月1日后的所有事件。

```
runoob@runoob:~/mysql$ docker events  --since="1467302400"
2016-07-08T19:44:54.501277677+08:00 network connect 66f958fd13dc4314ad20034e576d5c5eba72e0849dcc38ad9e8436314a4149d4 (container=b8573233d675705df8c89796a2c2687cd8e36e03646457a15fb51022db440e64, name=bridge, type=bridge)
2016-07-08T19:44:54.723876221+08:00 container start b8573233d675705df8c89796a2c2687cd8e36e03646457a15fb51022db440e64 (image=nginx:latest, name=elegant_albattani)
2016-07-08T19:44:54.726110498+08:00 container resize b8573233d675705df8c89796a2c2687cd8e36e03646457a15fb51022db440e64 (height=39, image=nginx:latest, name=elegant_albattani, width=167)
2016-07-08T19:46:22.137250899+08:00 container die b8573233d675705df8c89796a2c2687cd8e36e03646457a15fb51022db440e64 (exitCode=0, image=nginx:latest, name=elegant_albattani)
...
```

显示docker 镜像为mysql:5.6 2016年7月1日后的相关事件。

```
runoob@runoob:~/mysql$ docker events -f "image"="mysql:5.6" --since="1467302400" 
2016-07-11T00:38:53.975174837+08:00 container start 96f7f14e99ab9d2f60943a50be23035eda1623782cc5f930411bbea407a2bb10 (image=mysql:5.6, name=mymysql)
2016-07-11T00:51:17.022572452+08:00 container kill 96f7f14e99ab9d2f60943a50be23035eda1623782cc5f930411bbea407a2bb10 (image=mysql:5.6, name=mymysql, signal=9)
2016-07-11T00:51:17.132532080+08:00 container die 96f7f14e99ab9d2f60943a50be23035eda1623782cc5f930411bbea407a2bb10 (exitCode=137, image=mysql:5.6, name=mymysql)
2016-07-11T00:51:17.514661357+08:00 container destroy 96f7f14e99ab9d2f60943a50be23035eda1623782cc5f930411bbea407a2bb10 (image=mysql:5.6, name=mymysql)
2016-07-11T00:57:18.551984549+08:00 container create c8f0a32f12f5ec061d286af0b1285601a3e33a90a08ff1706de619ac823c345c (image=mysql:5.6, name=mymysql)
2016-07-11T00:57:18.557405864+08:00 container attach c8f0a32f12f5ec061d286af0b1285601a3e33a90a08ff1706de619ac823c345c (image=mysql:5.6, name=mymysql)
2016-07-11T00:57:18.844134112+08:00 container start c8f0a32f12f5ec061d286af0b1285601a3e33a90a08ff1706de619ac823c345c (image=mysql:5.6, name=mymysql)
2016-07-11T00:57:19.140141428+08:00 container die c8f0a32f12f5ec061d286af0b1285601a3e33a90a08ff1706de619ac823c345c (exitCode=1, image=mysql:5.6, name=mymysql)
2016-07-11T00:58:05.941019136+08:00 container destroy c8f0a32f12f5ec061d286af0b1285601a3e33a90a08ff1706de619ac823c345c (image=mysql:5.6, name=mymysql)
2016-07-11T00:58:07.965128417+08:00 container create a404c6c174a21c52f199cfce476e041074ab020453c7df2a13a7869b48f2f37e (image=mysql:5.6, name=mymysql)
2016-07-11T00:58:08.188734598+08:00 container start a404c6c174a21c52f199cfce476e041074ab020453c7df2a13a7869b48f2f37e (image=mysql:5.6, name=mymysql)
2016-07-11T00:58:20.010876777+08:00 container top a404c6c174a21c52f199cfce476e041074ab020453c7df2a13a7869b48f2f37e (image=mysql:5.6, name=mymysql)
2016-07-11T01:06:01.395365098+08:00 container top a404c6c174a21c52f199cfce476e041074ab020453c7df2a13a7869b48f2f37e (image=mysql:5.6, name=mymysql)
```

如果指定的时间是到秒级的，需要将时间转成时间戳。如果时间为日期的话，可以直接使用，如--since="2016-07-01"。

### 6、logs

**docker logs :** 获取容器的日志

### 语法

```
docker logs [OPTIONS] CONTAINER
```

OPTIONS说明：

- **-f :** 跟踪日志输出
- **--since :**显示某个开始时间的所有日志
- **-t :** 显示时间戳
- **--tail :**仅列出最新N条容器日志

### 实例

跟踪查看容器mynginx的日志输出。

```
runoob@runoob:~$ docker logs -f mynginx
192.168.239.1 - - [10/Jul/2016:16:53:33 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.93 Safari/537.36" "-"
2016/07/10 16:53:33 [error] 5#5: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 192.168.239.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "192.168.239.130", referrer: "http://192.168.239.130/"
192.168.239.1 - - [10/Jul/2016:16:53:33 +0000] "GET /favicon.ico HTTP/1.1" 404 571 "http://192.168.239.130/" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.93 Safari/537.36" "-"
192.168.239.1 - - [10/Jul/2016:16:53:59 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.93 Safari/537.36" "-"
...
```

查看容器mynginx从2016年7月1日后的最新10条日志。

```
docker logs --since="2016-07-01" --tail=10 mynginx
```

### 7、wait

**docker wait :** 阻塞运行直到容器停止，然后打印出它的退出代码。

### 语法

```
docker wait [OPTIONS] CONTAINER [CONTAINER...]
```

### 实例

```
docker wait CONTAINER
```

### 8、export

**docker export :**将文件系统作为一个tar归档文件导出到STDOUT。

### 语法

```
docker export [OPTIONS] CONTAINER
```

OPTIONS说明：

- **-o :**将输入内容写到文件。

### 实例

将id为a404c6c174a2的容器按日期保存为tar文件。

```
runoob@runoob:~$ docker export -o mysql-`date +%Y%m%d`.tar a404c6c174a2
runoob@runoob:~$ ls mysql-`date +%Y%m%d`.tar
mysql-20160711.tar
```



### 9、port

**docker port :**列出指定的容器的端口映射，或者查找将PRIVATE_PORT NAT到面向公众的端口。

### 语法

```
docker port [OPTIONS] CONTAINER [PRIVATE_PORT[/PROTO]]
```

### 实例

查看容器mynginx的端口映射情况。

```
runoob@runoob:~$ docker port mymysql
3306/tcp -> 0.0.0.0:3306
```



## 三、仓库管理

### 1、login

**docker login :** 登陆到一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub

**docker logout :** 登出一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub

### 语法

```
docker login [OPTIONS] [SERVER]
docker logout [OPTIONS] [SERVER]
```

OPTIONS说明：

- **-u :**登陆的用户名
- **-p :**登陆的密码

### 2、pull

**docker pull :** 从镜像仓库中拉取或者更新指定镜像

### 语法

```
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

OPTIONS说明：

- **-a :**拉取所有 tagged 镜像

  

- **--disable-content-trust :**忽略镜像的校验,默认开启

  

### 实例

从Docker Hub下载java最新版镜像。

```
docker pull java
```

从Docker Hub下载REPOSITORY为java的所有镜像。

```
docker pull -a java
```

### 3、push

**docker push :** 将本地的镜像上传到镜像仓库,要先登陆到镜像仓库

### 语法

```
docker push [OPTIONS] NAME[:TAG]
```

OPTIONS说明：

- **--disable-content-trust :**忽略镜像的校验,默认开启

### 实例

上传本地镜像myapache:v1到镜像仓库中。

```
docker push myapache:v1
```

### 4、search

**docker search :** 从Docker Hub查找镜像

### 语法

```
docker search [OPTIONS] TERM
```

OPTIONS说明：

- **--automated :**只列出 automated build类型的镜像；
- **--no-trunc :**显示完整的镜像描述；
- **-s :**列出收藏数不小于指定值的镜像。

### 实例

从Docker Hub查找所有镜像名包含java，并且收藏数大于10的镜像

```
runoob@runoob:~$ docker search -s 10 java
NAME                  DESCRIPTION                           STARS   OFFICIAL   AUTOMATED
java                  Java is a concurrent, class-based...   1037    [OK]       
anapsix/alpine-java   Oracle Java 8 (and 7) with GLIBC ...   115                [OK]
develar/java                                                 46                 [OK]
isuper/java-oracle    This repository contains all java...   38                 [OK]
lwieske/java-8        Oracle Java 8 Container - Full + ...   27                 [OK]
nimmis/java-centos    This is docker images of CentOS 7...   13                 [OK]
```

参数说明：

**NAME:** 镜像仓库源的名称

**DESCRIPTION:** 镜像的描述

**OFFICIAL:** 是否 docker 官方发布

**stars:** 类似 Github 里面的 star，表示点赞、喜欢的意思。

**AUTOMATED:** 自动构建。



## 四、本地镜像管理

### 1、images

**docker images :** 列出本地镜像。

### 语法

```
docker images [OPTIONS] [REPOSITORY[:TAG]]
```

OPTIONS说明：

- **-a :**列出本地所有的镜像（含中间映像层，默认情况下，过滤掉中间映像层）；

  

- **--digests :**显示镜像的摘要信息；

  

- **-f :**显示满足条件的镜像；

  

- **--format :**指定返回值的模板文件；

  

- **--no-trunc :**显示完整的镜像信息；

  

- **-q :**只显示镜像ID。

  

### 实例

查看本地镜像列表。

```
runoob@runoob:~$ docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
mymysql                 v1                  37af1236adef        5 minutes ago       329 MB
runoob/ubuntu           v4                  1c06aa18edee        2 days ago          142.1 MB
<none>                  <none>              5c6e1090e771        2 days ago          165.9 MB
httpd                   latest              ed38aaffef30        11 days ago         195.1 MB
alpine                  latest              4e38e38c8ce0        2 weeks ago         4.799 MB
mongo                   3.2                 282fd552add6        3 weeks ago         336.1 MB
redis                   latest              4465e4bcad80        3 weeks ago         185.7 MB
php                     5.6-fpm             025041cd3aa5        3 weeks ago         456.3 MB
python                  3.5                 045767ddf24a        3 weeks ago         684.1 MB
...
```

列出本地镜像中REPOSITORY为ubuntu的镜像列表。

```
root@runoob:~# docker images  ubuntu
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              14.04               90d5884b1ee0        9 weeks ago         188 MB
ubuntu              15.10               4e3b13c8a266        3 months ago        136.3 MB
```