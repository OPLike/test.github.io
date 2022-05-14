# docker常用命令



## docker基本命令

##### 1：停止运行的容器

```sh
docker stop + id/name   example:  docker stop 123d612c8998/redis_bak
docker  rm $(sudo docker ps -a -q) //删除所有正在运行的镜像
```



##### 2：强制删除镜像(推荐停止镜像之后删除)

```shell
docker rmi -f + id/name   example: docker rmi -f 123d612c8998/redis_bak
docker rmi `docker images` //删除所有镜像
docker rmi `docker images --filter "dangling=true"` //删除所有的tag或者容器为none的
```



##### 3：打镜像 先准备一个Dockerfile 

```shell
FROM anapsix/alpine-java:8_server-jre_unlimited   <-- 表示所需要的运行环境 必须为第一句 命令开头要大写 比如FROM CMD  -->

RUN mkdir -p /dfs    <--运行 run命令创建 文件夹dfs run 之后的命令跟正常的shell命令一致-->

WORKDIR /dfs   <--将dfs目录设置为工作目录  WORKDIR 表示设置工作目录-->

ADD ./DFS.jar ./   <--将jar包放入工作目录-->

CMD java -Djava.security.egd=file:/dev/./urandom -jar DFS.jar --spring.config.location=application.yml <--CMD 表示这个镜像启动时会执行的命令-->
```

然后执行打镜像的命令

```shell
docker  build -t name:tag .  name为你想要打的docker 仓库的名字 tag为版本号 .表示的是打包目录为当前目录 也可以填dockefile的绝对路径
```



##### 4：将你打包好的镜像推送到仓库中

```shell
docker push name:tag   name表示的仓库的名字  tag为版本号 example: docker push hdwuhan.ddns.info:41857/dfs:1.8
```



##### 5：从仓库中拉取镜像

```shell
docker pull name:tag  example: docker pull hdwuhan.ddns.info:41857/dfs:1.8 私有仓库的话 可能需要配置daemon才能拉取
```



##### 6：登陆私有仓库

```shell
docker login hdwuhan.ddns.info:41857   需要指派一个私有仓库路径 如果不加的话默认登陆的是doker提供的公有仓库
```



##### 7：运行镜像

```shell
docker run -d -p 20000:9999 --name dfs_test_server hdwuhan.ddns.info:41857/dfs/dfs_test_server:1.6
20000表示对外暴露端口  9999表示内网连接端口 hdwuhan.ddns.info:41857/dfs/dfs_test_server表示仓库名 1.6是版本号  --name是运行之后给容器重命名
```



##### 8：查询docker hub上的相关安装包的镜像

```shell
docker search 服务名称  比如：docker search nginx
```



##### 9：查看容器运行日志

```shell
$ docker logs [OPTIONS] CONTAINER
  Options:
        --details        显示更多的信息
    -f, --follow         跟踪实时日志
        --since string   显示自某个timestamp之后的日志，或相对时间，如42m（即42分钟）
        --tail string    从日志末尾显示多少行日志， 默认是all
    -t, --timestamps     显示时间戳
        --until string   显示自某个timestamp之前的日志，或相对时间，如42m（即42分钟）
        
 查看指定时间后的日志，只显示最后100行：
  docker logs -f -t --since="2018-02-08" --tail=100 CONTAINER_ID
  查看最近30分钟的日志:
  docker logs --since 30m CONTAINER_ID
  查看某时间之后的日志：
  docker logs -t --since="2018-02-08T13:23:37" CONTAINER_ID
  查看某时间段日志：
  docker logs -t --since="2018-02-08T13:23:37" --until "2018-02-09T12:23:37" CONTAINER_ID   
```


​    



##### 10：设置容器日志大小 和日志文件个数

新建/etc/docker/daemon.json，若有就不用新建了。添加log-dirver和log-opts参数，样例如下：

```json
{
  "log-driver":"json-file",
  "log-opts": {"max-size":"500m", "max-file":"3"}  # max-size=500m，意味着一个容器日志大小上限是500M，max-file=3，意味着一个容器有三个日志，分别是id+.json、id+1.json、id+2.json。
}
```

//配置镜像加速

```json
{ 
    "registry-mirrors": ["http://hub-mirror.c.163.com","https://docker.mirrors.ustc.edu.cn"],
    "insecure-registries":["47.88.149.108:61010", "hdwuhan.ddns.info:41857", "harbor.iotwedora.com:40180"] 
}
```


设置之后重启docker守护进程 

```shell
systemctl daemon-reload 
systemctl restart docker
```

这个设置方法只对docker设置之后新增加的容器有效 对之前存在的docker容器无效

/etc/docker目录为docker安装的默认目录



##### 11：docker 创建有初始化数据的mysql镜像

```shell
Dockerfile

FROM mysql:5.5.62   //mysql版本
COPY ./dfs.sql /docker-entrypoint-initdb.d/  //将初始脚本拷贝到docker虚拟机中然后运行


```



##### 12：docker-compose 运行指定compose 文件

```html
1: 和docker-compose.yml 文件在同级目录下
    docker-compose up -d
2: 不在同一级目录
    docker-compose -f /home/docker/docker-compose.yml up -d
3:指定多个compose文件
        docker-compose -f /home/docker/docker-compose-1.yml -f /home/docker/docker-compose-2.yml up -d
```



##### 13：查看正在运行的容器网络状态及相关操作

```shell
docker network ls  //查看网络状态
docker network inspect test1 //查看test1网络的网络情况
docker network rm test1 test1-1 test-2 //将test1网络中的test1-1和test-2容器移出网络
docker network connect test1 container1 //将container1 加入到test1的网络中
```



##### 14：docker容器网络命名规则

```html
命名规则 ： COMPOSE_PROJECT_NAME_networks
COMPOSE_PROJECT_NAME 为docker容器的环境变量名称 可以在.env文件中设置 没有的话默认是docker-compose.yml所在文件夹的名称
比如 如果无COMPOSE_PROJECT_NAME 无networks 文件夹名为dfs_test 则网络名为 dfstest_default
如果COMPOSE_PROJECT_NAME=dfs 没有在docker-compose.yml文件中设置网络 则网络名称为dfs_default
docker-compose 整个容器的默认环境变量文件为同级目录下的.env 文件
```



##### 15：docker 容器内运行宿主机命令

```shell
docker run -it -d  \
-v /usr/bin/docker:/usr/bin/docker \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /usr/lib64/libltdl.so.7:/usr/lib/x86_64-linux-gnu/libltdl.so.7 镜像名称
docker run -d -p 6666:3306 --env MYSQL_ROOT_PASSWORD=root --env MYSQL_ALLOW_EMPTY_PASSWORD=sa --env MYSQL_RANDOM_ROOT_PASSWORD=sa --name mysql_test hdwuhan.ddns.info:41857/wedora/mysql:2.0_design_fields
```



##### 16: docker-compose linux 安装

将官网下载的docker-compose文件 放在/usr/local/bin目录下  并且授权 

```shell
chmod +x /usr/local/bin/docker-compose
```



##### 17：docker inspect 相关命令

```shell
//获取网络中的各个容器名称和ID
docker network inspect wedorastandard_default
//获取对应映射的外网端口
docker inspect --format='{{range $p, $conf := .NetworkSettings.Ports}} {{(index $conf 0).HostPort}} {{end}}' containetID
//查询容器名对应的内网IP
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' containetID
```



##### 18：docker  save load (把镜像文件保存 和离线加载镜像文件)

```shell
1：docker save -o wedora.tar.gz 03ab3311dc99 0d23b7632e8e  将ID为03ab3311dc99 和0d23b7632e8e的两个镜像保存为wedora.tar.gz文件
2：docker save  03ab3311dc99 0d23b7632e8e | gzip > wedora2.tar.gz 以gzip压缩的方式保存两种镜像 可以压缩的更小
3：docker load -i wedora.tar.gz 把打包好的镜像文件加载出来 就类似于从镜像仓库拉取
使用id打包的话再load到新换将的时候仓库名和tag值会为none 推荐在打包镜像的时候使用仓库名加上tag值 如hdwuhan.ddns.info:41857/wedora/wedora_server:2.2_cloudcover
```



##### 19：从docker宿主机进入容器内部

```shell
docker exec -it mysql_docker /bin/sh  进入mysql_docker 容器内部
```



##### 20：docker swarm

```shell
docker swarm init //创建swarm
docker swarm leave //离开节点
docker swarm join-token --rotate worker|manager 查询加入swarm worker或者manager 节点的命令
docker service ls //查看deploy启动的服务
docker service logs -f wedora 查看wedora这个服务的日志

docker stack deploy -c docker-stack.yml dfs --with-registry-auth
docker stack ls //查看stack部署网络
docker stack rm 网络名//删除网络
```



##### 21：docker 批量删除镜像

```shell
docker images|grep none|awk '{print $3}'|xargs docker rmi  //批量删除tag为none的镜像
```



##### 22：容器和宿主机之间的文件交互

```shell
docker cp wedora:/tmp/1.txt /tmp/1.txt  //将wedora容器内部的1.txt文件拷贝到宿主机中
docker cp /tmp/1.txt wedora:/tmp/1.txt //将宿主机中的1.txt 拷贝到容器里
```



##### 23：docker镜像重命名

```sh
docker  tag  镜像id  仓库：标签
或：
docker  tag  旧镜像名  新镜像名
```


docker tag docker.io/macintoshplus/rabbitmq-management rabbitmq



##### 24：在容器外让某个容器内部执行命令

```shell
docker exec -it e8ee6590ab3d /bin/sh -c "wget https://alibaba.github.io/arthas/arthas-boot.jar && java -jar arthas-boot.jar 1"
```





## docker操作postgresql容器



##### 进入pgsql容器

```shell
docker exec -it pgsql_iot /bin/bash
```



##### 备份指令

```shell
pg_dump -U postgres ops_hd > /pgdump/ops_hd.sql
```



## 全量备份数据库脚本

```shell
#!/bin/bash
time=$(date "+%Y%m%d%H")
str="pg_dump -U postgres ops_hd > /pgdump/ops_hd$(date "+%Y%m%d%H").sql"
docker exec pgsql_iot /bin/sh -c "pg_dump -U postgres ops_hd > /pgdump/ops_hd$(date "+%Y%m%d%H").sql"
docker cp pgsql_iot:/pgdump/ops_hd$(date "+%Y%m%d%H").sql /usr/ops_data/sql/
cp -r -u /home/ops/ops/data/web_server_ops /usr/ops_data
source /etc/profile
```

