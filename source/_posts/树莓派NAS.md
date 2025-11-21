---
title: 树莓派 NAS
date: 2023-11-15 10:10
tags: 配置
---

> 原文发表在[公众号](https://mp.weixin.qq.com/s/pOBAxPJq1To0dAsysxoTkw)

#### docker 常用命令

```apl
# 停止所有的容器
docker stop $(docker ps -aq)

# 删除所有的容器
docker rm $(docker ps -aq)

# 删除所有的镜像
docker rmi $(docker images -q)

# 删除所有不使用的镜像
docker image prune --force --all

# 删除所有停止的容器
docker container prune -f

# 清理未使用和悬空的镜像
docker image prune

# 只清理悬空的镜像
docker image prune -a

# 清理停止运行的容器
docker container prune

# 清理未使用的卷宗
docker volume prune
```

#### 安装Portainer 中文版 ip:9000

```apl
docker run -d --restart=always --name="portainer" -p 8000:8000 -p 9443:9443 -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock 6053537/portainer-ce
```

#### 挂载硬盘

```apl
# 查看新添加的盘符
sudo fdisk -l

# 挂载硬盘到该文件夹上
sudo mount /dev/sda1 /mnt/sda1

# 查询挂载硬盘UUID
sudo blkid /dev/sda1

# 设置开机挂载硬盘
sudo vim /etc/fstab
# UUID=******   /mnt/sda1       ntfs   defaults       0       1
[UUID=******][挂载硬盘分区] [挂载硬盘格式] 0 2
第一个数字：0表示开机不检查磁盘，1表示开机检查磁盘
第二个数字：0表示交换分区，1表示启动分区（Linux），2表示普通分区
```

#### 安装samba（共享硬盘）

```apl
docker run -it --name samba \
-p 445:445 \
-v /mnt/sdb1:/private \
-v /mnt/sdc1:/public \
-d dperson/samba \
-u "user;password" \
-g "aio read size = 0" \
-g "aio write size = 0" \
-r \
-s "private;/private/;yes;no;no;user;user" \
-s "public;/public/;yes;yes;no;all;user"
```

#### 安装netdata ip:19999

```apl
docker run -d --name=netdata \
 --pid=host \
 --network=host \
 -v netdataconfig:/etc/netdata \
 -v netdatalib:/var/lib/netdata \
 -v netdatacache:/var/cache/netdata \
 -v /etc/passwd:/host/etc/passwd:ro \
 -v /etc/group:/host/etc/group:ro \
 -v /proc:/host/proc:ro \
 -v /sys:/host/sys:ro \
 -v /etc/os-release:/host/etc/os-release:ro \
 -v /var/run/docker.sock:/var/run/docker.sock:ro \
 --restart unless-stopped \
 --cap-add SYS_PTRACE \
 --cap-add SYS_ADMIN \
 --security-opt apparmor=unconfined \
 netdata/netdata
```

#### Docker安装aria2

```apl
# 方式一
docker run -d \
 --name=aria2 \
 -e PUID=$UID \
 -e PGID=$GID \
 -e TZ=Asia/Shanghai \
 -e SECRET=****** \
 -e CACHE=512M \
 -e PORT=6800 \
 -e BTPORT=32516 \
 -e WEBUI=true \
 -e WEBUI_PORT=8080 \
 -e UT=true \
 -e RUT=true \
 -e FA=none \
 -e QUIET=true \
 -e SMD=true \
 -p 32516:32516 \
 -p 32516:32516/udp \
 -p 6800:6800 \
 -p 8080:8080 \
 -v /usr/aria2/config:/config \
 -v /mnt/sda1/download:/downloads \
 --restart always \
 superng6/aria2:webui-latest

# 方式二
docker run -d \
   --name aria2-pro \
   --restart unless-stopped \
   --log-opt max-size=1m \
   -e PUID=$UID \
   -e PGID=$GID \
   -e UMASK_SET=022 \
   -e RPC_SECRET=****** \
   -e RPC_PORT=6800 \
   -e FA=none \
   -p 6800:6800 \
   -e LISTEN_PORT=6888 \
   -p 6888:6888 \
   -p 6888:6888/udp \
   -v /usr/aria2/config:/config \
   -v /mnt/sda1/download:/downloads \
   p3terx/aria2-pro

###
### https://p3terx.com/archives/docker-aria2-pro.html

###
启动容器命令参数详解
用户和组设定：
PUID=$UID、PGID=$GID这2个定义用户和用户组的环境变量，限定了aria2以什么用户和用户组运行，不指定则默认使用nobady用户和nogroup用户组，但在使用FileRun网盘时，会因权限问题无法删除或改名aria2下载好的文件，所以PUID和GUID要指定为和WEB环境的运行用户和用户组一致，比如WEB环境运行的用户及对应的用户组都是WWW，对应的uid和gid都是1001，那就要指定PUID=1001、PGID=1001，这样在FileRun网盘中就可以正常的进行删除和修改操作了；
几个环境变量:
-e UMASK_SET=022 ，设置umask，默认值022；
-e RPC_SECRET=，设置RPC密钥，用于AriaNg与Aria2的通讯验证使用；
-e RPC_PORT=6800，设置PRC通讯端口（与宿主主机的端口映射一致）；
-e LISTEN_PORT=6888，BT 监听端口（TCP）、DHT 监听端口（UDP）设置，即 Aria2 配置中listen-port与dht-listen-port选项定义的端口。如果没有设置，配置文件中的默认值为6888。
容器目录挂载，将/downloads挂载到宿主主机的/root/aria2/downloads:/downloads目录，即FileRun的数据目录中，方便下载完成直接在网盘中查看；配置文件挂载到指定的宿主主机目录/root/aria2/config中，宿主主机的目录根据实际情况自行修改；
3个端口映射：
-p 16800:6800，为RPC 通讯端口映射；
-p 16888:6888，为BT 监听端口（TCP）映射，即 Aria2 配置中listen-port选项定义的端口；
-p 16888:6888/udp，为DHT 监听端口（UDP）映射，即 Aria2 配置中dht-listen-port选项定义的端口。
```

#### Docker安装AriaNg

```apl
docker run -d \
   --name ariang \
   --log-opt max-size=1m \
   -p 6880:6880 \
   --restart always \
   p3terx/ariang
```

