## 0.写在安装之前

希望安装之前对于docker有一定的初步了解（https://www.runoob.com/docker/docker-tutorial.html），对于linux服务器有一些基本了解。

对于宿主机需要安装必要的库

```
sudo yum install epel-release
sudo yum install -y yum-utils mysql psmisc file which net-tools wget unzip telnet git gcc gcc-c++ make glibc-devel flex bison ncurses-devel protobuf-devel zlib-devel openssl-devel  mariadb-devel.x86_64 libmemcached-devel.x86_64 jsoncpp-devel.x86_64 apr-util-devel.x86_64 libcurl-devel.x86_64 python3
pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple zmail requests pymysql oss2
```



## 1.安装docker-ce

参考安装方法：https://developer.aliyun.com/article/110806

本次安装机器为 Ubuntu 16.04.4 LTS

```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

注：可能直接执行会报下面的错误
# Executing docker install script, commit: 7cae5f8b0decc17d6571f9f52eb840fbc13b2737
+ sudo -E sh -c 'yum install -y -q yum-utils'
Sorry, user qwprd is not allowed to execute '/bin/sh -c yum install -y -q yum-utils' as root on WTCCN-SA-QWGW1.

则执行
sudo curl -fsSL https://get.docker.com >a.sh
sudo vi a.sh
将287行的sudo -E sh -c 改为 sudo sh -c
sudo chmod +x a.sh 
sudo ./a.sh 
```


并且设置docker镜像加速

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://l9jy5ong.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

PS:有时候在亚马逊的服务器上面安装centos比较慢，可以更新centos软件源，仅仅针对centos机器。

```
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
yum makecache
```

PS:如果系统时区不对，请先调整系统时区

```
sudo cp  /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
```

## 2.安装mysql

一般都是采用阿里云mysql，则跳过这一步，如果要在本地安装mysql，参考 ：https://blog.csdn.net/weixin_42209572/article/details/98983741



## 3.安装tars框架

docker镜像地址：https://hub.docker.com/r/tarscloud/framework/tags?page=1&ordering=last_updated

有两种 docker 可用:

- framework: Tars 框架 Docker 制作脚本, 制作的 docker 包含了框架核心服务和 web 管理平台

- tars: Tars 框架 Docker 制作脚本, 和 framework 比, 增加了 java, nodejs 等运行时支持, 即可以把 java, nodejs 服务发布到 docker 里面(docker 里面安装了 jdk, node, php 环境)

  

本次选取的最新发布的稳定版本v2.4.17


```
sudo docker pull tarscloud/framework:v2.4.17
```



根据数据库配置framework框架脚本

```
sudo docker run -d \
    --name=tars-framework \
    --net=host \
    --restart=always \
    -e MYSQL_HOST="172.25.0.2" \
    -e MYSQL_ROOT_PASSWORD="123456" \
    -e MYSQL_USER=root \
    -e MYSQL_PORT=3306 \
    -e REBUILD=false \
    -e INET=eth0 \
    -e SLAVE=false \
    -v /data/framework:/data/tars \
    -v /data/framework/weblog:/usr/local/tars/cpp/deploy/web/log \
    -v /etc/localtime:/etc/localtime \
    -p 3000:3000 \
    -p 3001:3001 \
    tarscloud/framework:v2.4.17
```

MYSQL_IP: mysql 数据库的 ip 地址

MYSQL_ROOT_PASSWORD: mysql 数据库的 root 密码

INET: 网卡的名称(ifconfig 可以看到, 比如 eth0), 表示框架绑定本机 IP, 注意不能是 127.0.0.1

REBUILD: 是否重建数据库,通常为 false, 如果中间装出错, 希望重置数据库, 可以设置为 true

SLAVE: 是否是从节点, 可以部署多台机器, 通常一主多从即可.

MYSQL_USER: mysql 用户, 默认是 root

MYSQL_PORT: mysql 端口

-v /data/framework:/data/tars  是将容器内的/data/tars映射到宿主机的/data/framework目录



注：MYSQL_ROOT_PASSWORD不能为空字符串



PS:可以通过docker命令查看日志

```
sudo docker logs tars-framework
```



如果在安装过程中mysql提示没有授权，可以尝试手动授权一下

```
CREATE USER 'tarsAdmin'@'%' IDENTIFIED WITH mysql_native_password BY 'Tars@2019';
GRANT "SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE USER" ON *.* TO 'tarsAdmin'@'%' WITH GRANT OPTION;

CREATE USER 'tarsAdmin'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Tars@2019';
GRANT "SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE USER" ON *.* TO 'tarsAdmin'@'localhost' WITH GRANT OPTION;

CREATE USER 'tarsAdmin'@'{HOSTIP}' IDENTIFIED WITH mysql_native_password BY 'Tars@2019';
GRANT "SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE USER" ON *.* TO 'tarsAdmin'@'{HOSTIP}' WITH GRANT OPTION;
```



## 4.安装tarsnode节点

仅仅在需要的机器上面安装，并且不能和framework的机器相同

docker镜像地址：https://hub.docker.com/r/tarscloud/tars-node/tags?page=1&ordering=last_updated

```
最新版本
sudo docker pull tarscloud/tars-node:latest

稳定版本
sudo docker pull tarscloud/tars-node:stable

全版本
sudo docker pull tarscloud/tars-node:full
```

```
sudo docker run -d \
    --name=tars-node \
    --net=host \
    --restart=always \
    -e INET=eth0 \
    -e WEB_HOST="http://管理平台ip:3000" \
    -v /data/node:/data/tars \
    -v /etc/localtime:/etc/localtime \
    tarscloud/tars-node:stable
```

进入容器更新库

```
sudo docker exec -ti tars-node /bin/bash
```



```
yum install -y yum-utils mysql psmisc file which net-tools wget unzip telnet git gcc gcc-c++ make glibc-devel flex bison ncurses-devel protobuf-devel zlib-devel openssl-devel  mariadb-devel.x86_64 libmemcached-devel.x86_64 jsoncpp-devel.x86_64 apr-util-devel.x86_64 libcurl-devel.x86_64 python3
pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple zmail requests pymysql oss2
```



## 5.判断服务是否安装完毕

1、根据docker状态判断

```
sudo docker ps -a
```

查看名称为tarscloud/framework的image的运行状态

2、查看3000端口是否被监听

```
netstat -anpl | grep 3000
```

3、到/data/framework映射的目录查看对应日志文件

------

PS：如何进入到容器中呢，可以先用过

```
sudo docker ps -a
```

查到得到容器的名称或者CONTAINER ID

> CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS               NAMES
> 4653d9a7992f        tarscloud/framework:v2.4.6   "/usr/local/tars/cpp…"   11 hours ago        Up 11 hours                             tars-framework

可以使用container id或者names直接进入

```
sudo docker exec -ti tars-framework /bin/bash
```



## 6.设置管理平台用户密码

访问部署机器ip:3000地址，首次登陆需要设置密码，后续使用密码进行登陆.

至此我们已经完成了框架部署，接下来我们需要完成自己的服务部署。



## 7.告警系统安装

请参考tars-alarm-docker目录下面的文档完成



## 10.清理脚本安装

请参考clean-script目录下面的文档完成



## 11.其他修改项

### 数据库修改项

修改interactive_timeout，wait_timeout为60。修改位置为云数据库的参数配置中

### 宿主机修改项

如果有产生core文件，需要在宿主机上面设置core文件路径

```
echo "/data/tars/app_log/core.%e.%p"  | sudo tee /proc/sys/kernel/core_pattern
```



## 12.业务请求时遇到的问题和解决办法



### 1.nginx转发请求不到gateway上

两台内网机器配置了域名80端口，需要将请求转发到GatewayServer上，nginx配置如下：

server
{
    listen 80;
    server_name epwai.watsonsvip.com.cn;
    location /comm/ {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://10.44.2.56:8200/;  #本机ip和gateway的ProxyObj servant的端口号
    }
}

### 2.Gateway转发时报错

报错信息如下，请求没有转发到对应的服务上
```
2021-05-26 10:16:14|139841027979136|DEBUG|[TARS]accept [10.44.2.56:56146] [47] incomming
2021-05-26 10:16:14|139840226055936|INFO|[TARS]processAuth no need auth func, auth succ
2021-05-26 10:16:14|139840804857600|ERROR|[ServantHandle::handle request protocol decode error:require field not exist, tag: 1, headTag: 0]
```
将GatewayServer的ProxyObj servant 配置改为协议类型改为“**非tars**”

![image-20210526103132618](C:\Users\johnson\AppData\Roaming\Typora\typora-user-images\image-20210526103132618.png)

