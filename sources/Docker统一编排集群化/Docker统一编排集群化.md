---
author: Dbin
startDate: 2022-11-13
endDate: null
---

[TOC]

## 工程时间4 课题一 学习记录(前言)

1. 这里是前言,你可以不必一个一个点进去观看,可以大概浏览,选择一些使用可以直接从[安装Docker](#install_docker)开始

2. 本记录使用Ubuntu20.04 虚拟机,记录中💖比*重要。

3. 一些资料:
   1. [Docker简介](https://www.jianshu.com/p/631605b266f8)
   2. [什么是docker 容器编排)](https://www.west.cn/docs/61366.html)
   3. [Swarm 集群管理 | 菜鸟教程 (runoob.com)](https://www.runoob.com/docker/docker-swarm.html)
   4. [Docker Compose | 菜鸟教程 (runoob.com)](https://www.runoob.com/docker/docker-compose.html)

4. 工具或资源推荐

   1. 

      - 虚拟机工具Vmware,
      - 🪜推荐1:[Watt Toolkit](https://steampp.net/) (提供访问github的服务)
      - 🪜推荐2:[clash](https://github.com/Dreamacro/clash/tags) (工具,并未提供服务,服务请自行寻找)
      - 

   2. 阿里云镜像网站：[阿里巴巴开源镜像站-OPSX镜像站-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/mirror/)

   3. ssh工具(任选其一):

      1. [MobaXterm Xserver with SSH](https://mobaxterm.mobatek.net/download.html)(<font color='blue'>推荐</font>)
      2. [Birvise SSH Client](https://www.bitvise.com/) (<font color='grey'>需要科学上网</font>)
      3. [Micorsoft Store 终端(Windows Terminal)](https://apps.microsoft.com/store/detail/windows-terminal/9N0DX20HK701?hl=zh-cn&gl=cn) (这里默认使用的是powershell)

      如果用普通终端命令链接linux主机的话,这里有命令:

       1. 密码链接:

          ```shell
          ssh  root@114.132.58.142  -p 22
          ```

          其中root是用户名,@后面是主机ip, -p后面是端口,之后输入密码即可远程登录,前提是主机有ssh服务.

          [ linux安装ssh和开启 ssh服务](https://blog.csdn.net/java_dotar_01/article/details/76942563)

       2. 使用私钥链接

          ```shell
          ssh  root@114.132.58.142  -p 22 -i "这里是私钥路径,默认为C:\Users\user\.ssh\id_rsa" 
          ```

          -i指定密钥,若无则默认C:\Users\user\.ssh\id_rsa(C为系统盘);

   4. 踩坑提示

      1. 如果你和我一样采用Ubuntu20.04(阿里云的镜像(指的是ubuntu20.4可能会出现该问题),那么请运行`df -h`命令,查看还可以使用的磁盘空间是多少

         如果发现磁盘空间大小与分配的虚拟机空间大小不一致,可能是安装Ubuntu Server的时候采用了LVM

         请参考[磁盘满了问题解决](https://blog.csdn.net/Fish_Sheep/article/details/103325378)解决问题,趁磁盘空间足够的时候解决,别问我为什么。
         
         主要命令是这两条(请不要直接运行,具体操作看链接)
         
         ```shell
         lvextend -L 18G /dev/mapper/ubuntu--vg-ubuntu--lv #将容量扩容到18G
         resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv # 确认
         ```
         
         

## 0.安装虚拟机

### 1.安装

略......

<font color='blue'>tips</font> 强烈建议在安装时安装openSSH

### <a id='set_static_ip'>2.新增网卡配置静态ip</a>

此处配置作用主要应对网络变化,以免造成奇奇怪怪的问题(比如你设置建立好的集群,因为ip变化所有节点都脱离了管理节点,再给老师检查的时候出现这种情况就寄了),不过这里依旧可以跳过。或者你可以根据[准备工作](#VMware虚拟机克隆ip相同) 也将NAT设置为静态IP。

如果你和我一样配置静态ip，原本的NAT就仅用于真机远程连接虚拟机使用了,虚拟机之间通信(比如:节点加入集群的时候)就使用新添加的网卡通信,保证ip变动后节点不会脱离集群，最终要达成的网络结构是这样的

![t](./assets/t.png)

如果不配置静态ip，网络结构就是这样的

![t2](./assets/t2.png)

再VMware中添加一张网卡,如下图,由于种种原因,我的子网ip为192.168.71.0(之后两个虚拟机的ip都需要再这个网段,才可以相互连接)

![image-20221130155230481](./assets/image-20221130155230481.png)



为准备的虚拟机(两台虚拟机)添加网卡,其中NAT模式网卡主要用于和真机通信(真机远程控制)，自定义网卡用于虚拟机之间通信,且通过静态IP保证实验能够再迁移虚拟机后正常通信。

![image-20221130155534010](./assets/image-20221130155534010.png)

打开虚拟机后,运行`ip addr`可以看到两张网卡

![image-20221130160346198](./assets/image-20221130160346198.png)

我这里是一张ens33,一张ens37,其中ens33是NAT模式网卡,自动分配了ip,ens37是刚才添加的网卡(没有分配ip)。如果你同时开启两个虚拟机(其中一个由另一个克隆而来)遇到NAT模式网卡ip相同的问题,请参照[ VMware虚拟机克隆ubuntu20.04系统IP相同](https://blog.csdn.net/anlunson/article/details/124942722)修改,这里建议先修改一个虚拟机,之后克隆后修改一下就好。

执行以下命令修改网卡配置(其中`00-installer-config.yaml` 可能是其他名字,请自行查看)

```shell
sudo vim /etc/netplan/00-installer-config.yaml
```

我这里原本是这样的

```
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens33:
      dhcp4: true
  version: 2
```

改为：

```yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens33:
      dhcp4: true
    # ens37是新添加的
    # 网卡名,请注意对应先前查看的网卡
    ens37:
      # 静态ip
      dhcp4: false
      # ip地址与先前VMware网络编辑器中设置在同一个网段。
      addresses: [192.168.71.10/24]
      optional: true
  version: 2
```

执行`sudo netplan apply` 应用配置,再输入`ip addr`命令就可以查看到end37网卡 ip设置好了,另外一个虚拟机可以在克隆之后([准备工作](#DockerSwarmPrepare)之后)再设置,也是同样的操作,不过ip需要最后一位有所不同,我设置的是192.168.71.11,都配置好后,两台机器可以相互ping通就成功了(可能防火墙会导致ping失败)

## <a id='install_docker'>1.安装docker💖</a>

参考资料[Ubuntu Docker 安装 | 菜鸟教程 (runoob.com)](https://www.runoob.com/docker/ubuntu-docker-install.html#:~:text=Ubuntu Docker 安装 1 使用官方安装脚本自动安装 安装命令如下： curl -fsSL,Docker Engine-Community 的边缘版本和测试版本。 脚本的源代码在 docker-install 仓库中。 不建议在生产环境中使用这些脚本，在使用它们之前，您应该了解潜在的风险： )

在linux主机运行命令：

```shell
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

使用命令`docker info`或者`dockers version`查看当前docker信息

出现类似下列信息则说明安装成功

```shell
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  app: Docker App (Docker Inc., v0.9.1-beta3)
  buildx: Docker Buildx (Docker Inc., v0.9.1-docker)
  compose: Docker Compose (Docker Inc., v2.12.2)
  scan: Docker Scan (Docker Inc., v0.21.0)

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 20.10.21
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 1
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 io.containerd.runtime.v1.linux runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 1c90a442489720eec95342e1789ee8a5e1b9536f
 runc version: v1.1.4-0-g5fd4c4d
 init version: de40ad0
 Security Options:
  apparmor
  seccomp
   Profile: default
 Kernel Version: 5.4.0-131-generic
 Operating System: Ubuntu 20.04 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 2
 Total Memory: 2.825GiB
 Name: ubuntu
 ID: 2YIJ:5LAR:K425:QFYS:5VH2:UV6U:RMEY:MT7Y:6WWZ:5M26:GD7E:RRNV
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false
```

若出现以下Error信息，说明当前存在权限问题：

```shell
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  app: Docker App (Docker Inc., v0.9.1-beta3)
  buildx: Docker Buildx (Docker Inc., v0.9.1-docker)
  compose: Docker Compose (Docker Inc., v2.12.2)
  scan: Docker Scan (Docker Inc., v0.21.0)

Server:
ERROR: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/info": dial unix /var/run/docker.sock: connect: permission denied
errors pretty printing info
```

执行下面三步:

1. ```shell
   su #切换用户到超管
   ```

2. ```shell
   $ sudo gpasswd -a username docker   #将普通用户username加入到docker组,注意,username为当前用户名
   ```

3. ```shell
   $ newgrp docker  #更新docker组
   ```

后再运行`docker info` 即正常运行。

## <del>2.更换镜像源*</del>

tips:此阶段只为了下载镜像时更快,更换docker镜像源为阿里云镜像源,<font color='blue'>需要阿里账号</font>。

- 登录到[容器镜像服务 (aliyun.com)](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)
- 再镜像工具中的镜像加速器中申请镜像链接。
- 根据阿里云的操作文档 `2.配置镜像加速器` 配置镜像源

## 3.使用portainer*

这一大节可以暂时跳过,之后再装,影响不大.

参考资料:[Docker 管理工具 Portainer - 腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/2067415)

### 1.安装运行portainer

1. 执行命令,查看相关镜像

   ```shell
   docker search portainer
   ```

2. 拉取portainer(如果更换了阿里云的镜像源的话,下载应该比较快)

   ```shell
   docker pull portainer/portainer-ce
   ```

3. 运行命令

   ```shell
   docker run -d -p 9000:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --name prtainer-test portainer/portainer-ce
   ```

   本命令运行一个docker容器,并该容器的9000端口与linux主机的9000端口产生映射,访问linux主机端口即访问到docker容器的对应的端口.

   比如当前linux主机是虚拟机,容器则可以理解为虚拟机中运行的虚拟机,将虚拟机运行的虚拟机的一个端口映射到虚拟机的一个端口,可以通过访问虚拟机的ip+port访问该容器运行的服务。

### 2.使用portainer

#### 1.准备工作:

查看虚拟机防火墙是否开放了对应端口等

centOS相关命令:

```shell
systemctl status firewalld.service #防火墙状态
firewall-cmd --list-all #查看防火墙开放端口
firewall-cmd --permanent --add-port=3306/tcp #开启3306端口
firewall-cmd --reload #重启
```

Ubuntu相关命令:

```shell
sudo ufw status #查看防火墙状态
sudo ufw enable #开启防火墙
sudo ufw disable #关闭防火墙
sudo ufw version #查看防火前版本
sudo ufw default allow #默认允许外部访问本机
sudo ufw default deny #拒绝外部访问本机
sudo ufw allow 53 #允许外部访问53端口
sudo ufw deny 53 #拒绝外部访问53端口
sudo ufw allow from 192.168.0.1 #允许某个IP地址访问本机所有端口
```

一般先查看防火前状态,如果是关闭的就不管了(如果是具有公网ip的linux服务器,请开启,否则会暴露在危险之中)

如果防火墙开启状态,请打开对应需要访问的端口。

<font color='blue'>tips</font>:如果你的linux主机拥有图形界面,可直接在当前虚拟主机中访问,不用考虑防火墙问题,但是在做容器编排时仍需要关心防火墙问题。

#### 2.访问portainer

1. 查看linux虚拟主机ip

   ```shell
   ip addr
   ```

   可以看到几张网卡(包含虚拟网卡的ip地址),选择你安装系统时设置的对应网卡的ip来进行访问,我这里时`ens33`,IP是`192.168.231.135`

2. 直接真机访问查看的ip(浏览器访问)

   可以看到这个页面

   ![image-20221113114906867](./assets/image-20221113114906867.png)

#### 3.配置portainer

首次登录需要设置admin的密码,建议设置12345678,方便,设置复杂密码请牢记。

其中`Allow collection of anonymous statistics. You can find more information about this in our privacy policy).` 的意思是 `允许收集匿名统计信息。您可以在我们的[隐私政策](https://www.portainer.io/documentation/in-app-analytics-and-privacy-policy/)中找到有关此内容的更多信息。`可以取消勾选

我的版本是自动登录。登录后可以就看到本地运行的环境了

![image-20221113115608806](./assets/image-20221113115608806.png)

若无可以点击Environment连接到docker环境。注意连接本地环境url为`/var/run/docker.sock`，若要连接一个已存在的远程环境则为对应ip+port，此处是本地环境。

<font color='blue'>Tips:当完成了以上内容,建议使用VMware的快照功能保留当前进度.</font>

## 4.Docker Swarm 的部署和使用💖

<font color='blue'>Tips:开始之前,建议使用VMware的快照功能保留当前进度.</font>

### <a id='DockerSwarmPrepare'>1.准备工作</a>

1. 将刚才准备好的虚拟机(我称之为`manager`机,对应考核标准的`master`机)关机,

2. manager机克隆一份，作为工作节点(我称之为`node`机,对应考核标准上的`node`机)。

3. <font color='red'>***注意***</font>:之后的操作需要一定要分清楚是在那个主机上运行命令。

4. 打开manager机,`ip addr` 命令查看ip(我的是`192.168.231.135` )

5. 打开`node`机,使用`ip addr` 命令查看当前ip,可能当前虚拟机的ip与`manager`的ip是一样的可以参照[ VMware虚拟机克隆ubuntu20.04系统IP相同](https://blog.csdn.net/anlunson/article/details/124942722)修改。<a id='VMware虚拟机克隆ip相同'> </a> 如果你和我一样设置了静态ip，那么,需要按照[配置静态ip](#set_static_ip) 添加另外一张网卡的内容

6. 建议修改新可浓虚拟机的hostname方便区分

   ```shell
   sudo vim /etc/hostname
   ```

   

### 2.创建swarm集群管理节点

在`manager`主机上使用命令：

```shell
docker swarm init --advertise-addr 192.168.231.135 #此处ip为manager主机ip
```

<font color='red'>注意:</font>这里的IP是NAT模式下的ip,如果已经像前面[配置静态ip](#set_static_ip) 一样配置了多网卡静态ip,最好直接使用静态ip,使用的是ip地址为192.168.71.10的虚拟机作为管理节点,ip就是192.168.71.10

会得到一个这样的输出,证明初始化成功,copy保存该输出,之后有用。(不同的ip会有不同的输出,所以若你采用静态ip也可以直接复制,不用特意更改)

```shell
docker swarm join --token SWMTKN-1-1uhjzhask1qoa08svoywmfclz1gvvyzabkh67mw8znh5217e59-98yiya06arxn5pkra3h1pomnp 192.168.231.135:2377
```

如果很久以后你忘了上面的内容，可以通过以下命令查看

```shell
docker swarm join-token worker
```



### 3.创建swarm集群工作节点

1. 进入`node`机,运行以下命令:

   ```shell
   docker swarm join --token SWMTKN-1-1uhjzhask1qoa08svoywmfclz1gvvyzabkh67mw8znh5217e59-98yiya06arxn5pkra3h1pomnp 192.168.231.135:2377 #和上一步保存的一样。
   ```

   看到输出`This node joined a swarm as a worker.`则说明成功了。

   若无法成功建议检查是否被防火墙拦截.

2. 进入`manager`机,运行`docker info`命令。

   ![image-20221113142622722](./assets/image-20221113142622722-1668320784979-1.png)

   可以看到如图信息,manager指的是管理节点1个,Nodes指的是工作节点2个,其中一个是我们新加入的节点,另外一个是管理节点。

3. 查看节点信息

   ```shell
   docker node ls
   ```

   我这里已经运行了两个节点.

   ![image-20221113172636370](./assets/image-20221113172636370.png)

<font color='red'>***注意***</font>：跟集群管理有关的任何操作，都是在管理节点上操作的。

### 4.部署服务到集群中

此处使用一个helloworld服务随机指派给一个工作节点,在manager端运行以下命令

1. 首先拉取一个redis 镜像(如果docker已经换源了的话,拉取速度会比较快)：

   ```shell
   docker pull redis #默认latest版本，redis是一个非常6的高速缓存工具,常用于web后端,游戏等开发。
   ```

2. 运行以下命令查看是否拉取成功

   ```shell
   docker images 
   ```

   如果拉去失败可能是镜像源原因,或者空间不足,空间不足请参照[解决/var/lib/docker空间不足问题 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/266570357)解决,也可能是Ubuntu Server的时候采用了LVM,参考[(磁盘满了问题解决](https://blog.csdn.net/Fish_Sheep/article/details/103325378)解决

3. 随机指派一个节点运行redis

   `--replicas 1` 指的是分配任务到一个节点, 之后可以扩展

   `-name redis redis` 第一个redis指的是名字是redis,第二个指的是redis容器名

   ```shell
   docker service create --replicas 1 --name redis redis
   ```

4. 查看服务部署情况

   ```shell
   docker service ps redis
   ```

   

   ![image-20221113171820684](./assets/image-20221113171820684.png)

   可以看到这个redis是在ubuntu上,也就是本节点上.

   运行以下命令可以查看到部署的具体信息。

   ```shell
   docker service inspect --pretty redis
   ```

5. 扩展服务到两个节点(扩展到新节点需要下载镜像)

   ```shell
   docker service scale redis=2
   ```

   然后查看运行信息,可以发现redis已经运行在两个节点中。

   ![image-20221113172254900](./assets/image-20221113172254900.png)

6. 以下命令查询docker集群的所有服务

   ```shell
   docker service ls
   ```

### 5.删除服务

```shell
docker service rm redis
```

### 6.停止节点接收新任务

1. 使ubuntu停止接受新任务:

    ```shell
    docker node update --availability drain ubuntu
    ```

    ![image-20221113173148835](./assets/image-20221113173148835.png)

    ubuntu将进入Drain模式,此时部署redis,redis必定会部署到ubuntu2.

    即使通过`docker service scale redis=2`命令扩展也是扩展到ubuntu2.

2. 是Ubuntu重新接受任务

    ```shell
    docker node update --availability active ubuntu
    ```

    ![image-20221113173711258](./assets/image-20221113173711258-1668332232390-1.png)

### 7.脱离集群

输入`docker swarm leave` 可以脱离当前集群,但是这样输入会有警告,`docker swarm leave --force` 命令忽略警告。

### 8.others

可以通过之前安装的portainer管理服务。

![image-20221113174325186](./assets/image-20221113174325186.png)

![image-20221113174331899](./assets/image-20221113174331899.png) 

## 5.Docker Compose

资料:

​	[Docker Compose | 菜鸟教程 (runoob.com)](https://www.runoob.com/docker/docker-compose.html)或许你需要先学习yaml文件配置,对于前后端选手应该没啥问题。

资源：

​	[compose github 地址](https://github.com/docker/compose) 访问慢,当然也可以使用科学加速。

​	[GitCode 镜像地址](https://gitcode.net/mirrors/docker/compose) 访问更快,不过pull可能需要准备ssh密钥。

### 下载Compose

1. 下载版本v.2.12.2

    ```shell
    sudo curl -L "https://objects.githubusercontent.com/github-production-release-asset-2e65be/15045751/986b903d-8918-497f-afe5-dfb67249ad11?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20221113%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20221113T110811Z&X-Amz-Expires=300&X-Amz-Signature=2fabf821bec83e57de18a47f7fb897f69731d2bddf23ae59d579833c47497d22&X-Amz-SignedHeaders=host&actor_id=76925338&key_id=0&repo_id=15045751&response-content-disposition=attachment%3B%20filename%3Ddocker-compose-linux-x86_64&response-content-type=application%2Foctet-stream" -o /usr/local/bin/docker-compose
    ```

    下载到`/usr/local/bin/docker-compose`

    链接失效到github自行下载[Releases · docker/compose (github.com)](https://github.com/docker/compose/releases)

    我准备的下载[链接](https://dbin.shop/file/docker-compose)(2023年将失效)

2. 添加运行权限

    ```shell
    sudo chmod 755 /usr/local/bin/docker-compose
    ```

3. 创建软连接

    ```shell
    sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
    ```

4. 运行`docker-compose version`查看是否下载正确

5. 创建文件夹存放文件,我这里是`/home/dbin/compose`

    ```shell
    mkdir /home/dbin/compose
    cd /home/dbin/compose/
    ```
    
    现在compose就准备好了，compose目录就是之要用到的工作目录，不同的部署任务应该新建不同的目录,之后的操作除特殊说明都是在该目录下进行的。

### 使用Compose

使用compose部署一个简单的后端程序

1. 使用以下命令下载demo.jar，也可以直接访问链接下载demo.jar,如果该链接过期可以直接访问[这里](https://wwiq.lanzoub.com/i9LTq0ieafjc)下载，然后把文件复制到compose目录下。

   ```shell
   wget "http://f0.0sm.com/node0/2022/12/86395718DB6FF800-69565eafd6ca5174.jpg" -O demo.jar
   chmod 755 demo.jar # 赋权命令
   ```

   下载下来的demo.jar 是一个spring项目,默认监听8888端口。

   他的代码是这样的：

   `DemoApplication.class`

   ```java
   package com.example.demo;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   @SpringBootApplication
   @RestController
   public class DemoApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(DemoApplication.class, args);
       }
       @RequestMapping("/{message}")
       public String M(@PathVariable String message) {
           return message;
       }
       @RequestMapping("/**")
       public String M() {
           return "hello,If you visit 'host:port/hello', you can see hello";
       }
   }
   
   ```

   `application.properties`

   ```properties
   server.port = 8888
   ```

2. 使用vim(`vim Dockerfile`)新建Dockerfile输入以下内容

   ```text
   FROM java:8
   VOLUME /tmp
   ADD demo.jar /demo.jar
   RUN bash -c 'touch /demo.jar'
   EXPOSE 8888
   ENTRYPOINT ["java","-jar","demo.jar"]
   ```

   `FROM`:指的是从java:8镜像开始构建容器

   `ADD`:是将demo.jar 复制到容器的/demo.jar

   `RUN bash`:创建容器时运行命令

   `EXPOSE`:是通知 Docker 容器在运行时监听指定的网络端口

   `ENTRYPOINT`: 是运行的命令

   关于`CMD` `RUN` `ENTRYPOINT` 的区别可以查看[这里](https://www.jianshu.com/p/7d485b6346f7)

3. `vim  docker-compose.yml` 创建 docker-compose.yml

   输入以下类容

   ```yaml
   version: '2' # 表示该 Docker-Compose 文件使用的是 Version 2 file
   services:
     mytest:  # 指定服务名称为mytest
       image: "java:8" #指定镜像为java8
       build: .  # 指定 Dockerfile 所在路径，这里是当前路径
       ports:    # 指定端口映射,注意，前面的是主机端口，后面是容器端口。
         - "8889:8888"
   ```

4. 开始运行吧。

   两条命令选择一调即可(需要下载镜像，所以要等一会)

   ```shell
   docker-compose up # 该命令会进入到容器中
   docker-compose up -d # 该命令可以在后台运行
   ```

   我运行的是`docker-compose up`,就可以看到熟悉的spring项目启动日志啦，可以在倒数第四行发现他监听了8888端口，如果退出当前页面容器就会停止运行，如果使用第二条命令退出了就会继续运行

   ![image-20221211180722761](./assets/image-20221211180722761.png)

5. 体验一下

   我的虚拟机NAT模式网卡的地址是`192.168.159.128` 所以访问`http://192.168.159.128:8889/`就能看见内容了，访问呢`http://192.168.159.128:8889/helloworld`就可以看到helloworld的输出。

   到此，Compose的基本使用就结束了。

   你可以使用Docker基本命令查看停止容器的运行。

   ```shell
   docker ps -a # 查看所有的容器
   docker stop d567691cc14d # 停止id为d567691cc14d的容器
   docker rm d567691cc14d # 删除id为d567691cc14d的容器
   ```

   

