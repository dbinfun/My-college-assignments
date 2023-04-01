# 软件安装与卸载

## 安装

Ubuntu可以直接使用`apt-get`命令安装软件，但是，很多软件需要我们手动下载安装

目前，我接触到的各种软件安装一般有3种格式`.deb` 、 `.tar.gz`、`AppImage`

### deb格式

Ubuntu中安装包如果是`.deb`结尾，可以使用以下命令安装

```shell
dpkg -i *****.deb
```

也可以直接通过右键选择其他应用打开安装。

这种格式的文件，安装一般在`/opt/`目录下。

### tar.gz格式

如果安装包是`.tar.gz`结尾的，说明他是一个绿色软件，可以快速从一台电脑转移到另一台电脑运行,我们可以直接解压到喜欢的路径然后运行。不过,这样的应用程序仍然可能存在一些配置文件在家目录下的`.config`目录下,即`~/home/username/.config/`下

```shell
tar -zxvf ****.tar -C /***/***
```

其中：

- `z`: 用 gzip 压缩算法对归档文件进行压缩或解压缩操作。
- `x`: 提取（解压缩）归档文件。
- `v`: 显示正在进行的操作的详细信息，以便用户可以跟踪操作进展。
- `f`: 从归档文件中提取或创建归档文件。
- `-C`: 是解压到指定的目录

然后我们就可以从解压后的目录中找到`.sh`(一般是.sh结尾)文件用于运行软件。

### AppImage格式

这种格式的文件可以直接运行,(如navicat),同样也可能会在家目录下生成一些配置文件。

## 图标的创建

在Ubuntu中使用`deb`文件安装应用，一般可以在应用程序中找到图标,但是想要在桌面创建图标却不能直接拖动。Ubuntu桌面应用图标的格式是`.desktop`,下面是Ubuntu的桌面图标的源文件的主要类容

```properties
[Desktop Entry]
Version=2021.3
Type=Application
Name=name
Comment=Sophisticated text editor for code, markup and prose
Exec=/***/***.sh
Terminal=false
Icon=/***/***.svg
Categories=Development;
```

- Version=版本
- Type=Application
- Name=程序的名称
- Comment=这里应该是注释？
- Exec=程序的位置
- Terminal=是否要打开终端
- Icon=程序的图标
- Categories=类别可以通过分号分割，包括 `Utility`（实用工具）、`Development`（开发工具）、`Graphics`（图形应用程序）、`Internet`（互联网应用程序）等

通过上面的类容，我们在桌面新建`txt`文档，修改对应的应用路径，和图标,修改后缀为`.desktop`，使用赋权命令或者右键赋权后就可以双击运行了。

另外，在Ubuntu 20.04中，应用程序的desktop文件通常存储在以下几个文件夹中：

1. `/usr/share/applications` - 这个文件夹中存储了系统安装的所有应用程序的.desktop文件，包括软件中心中的应用程序。
2. `/usr/local/share/applications` - 这个文件夹中存储了本地安装的应用程序的.desktop文件。
3. `~/.local/share/applications` - 这个文件夹中存储了当前用户安装的应用程序的.desktop文件。

在收藏夹中的应用程序的desktop文件存储在以下位置：`~/.local/share/applications`

## 卸载

在Ubuntu中想要卸载一个软件可不像Windows一样可以从控制面板卸载，部分软件可以通过商店卸载，绿色软件可以直接删除本体和配置文件,使用`deb`格式安装的文件可以通过以下方式卸载。

1. 搜索软件，如搜索腾讯会议，其包名是`wemeet`,搜索meet就好了， 安装后在`/opt/`目录下可以找到，如果不知道包名，一般可以通过安装包或者软件名猜个大概

   ```shell
   dpkg -l | grep meet
   ```

   搜索之后可以看到具体的包名。

2. 卸载

   通过以下命令卸载

   ```shell
   sudo dpkg -r wemeet
   # 或者
   sudo apt purge wemeet
   ```

# 环境的配置

## apt-get

`apt-get`是Ubuntu中非常重要的命令

### 配置镜像

如果是配置阿里云镜像，可以在[阿里云镜像官网](https://developer.aliyun.com/mirror/ubuntu?spm=a2c6h.13651102.0.0.3a121b116vEgqZ)找到对应的配置方法，其他的同理。

另外，可以直接打开软件更新器，在设置的Ubuntu软件那一栏可以找到下载自的设置，默认就有阿里云的选项，手动选上即可。

### 主要命令

```shell
sudo apt-get install package_name #安装软件
sudo apt-get update #更新软件包列表
sudo apt-get upgrade #升级已安装软件包
sudo apt-get remove package_name #卸载软件包
sudo apt-get autoclean #清理无用软件包
sudo apt-get clean #清理已安装软件包缓存
apt-cache search keyword #查询软件包信息
apt-cache show package_name #显示已安装软件包信息 可以通过|grep过滤
```

## Java

### 下载Java

到[官网归档](https://www.oracle.com/cn/java/technologies/downloads/archive/)中可以下载历史版本的java，Linux下载tar.gz的文件解压即可

主要命令是

```shell
sudo tar -zxvf ****.tar -C /***/***
```

### 环境配置

```shell
sudo vim /etc/profile #编辑配置文件
```

添加或者修改配置文件

<font color='blue'>JAVA_HOME</font>:jdk目录，即前面的解压目录

```shell
JAVA_HOME=/usr/local/java/jdk1.8.0_221
CLASSPATH=%JAVA_HOME%/lib:%JAVA_HOME%/jre/lib
PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin
export PATH CLASSPATH JAVA_HOME
```

重载配置文件

```shell
sudo source /etc/profile
```

## NodeJs

NodeJS和npm可以直接通过apt-get安装

```shell
sudo apt-get install node
sudo apt-get install npm
```

## Maven

### 下载

到[Apache官网](https://maven.apache.org/download.cgi)下载想要的版本即可

### 配置镜像

按照[阿里云Maven仓库镜像官网](https://developer.aliyun.com/mvn/guide)配置指南配置即可

## Doker

安装阿里云官方的[教程安装docker-ce](https://developer.aliyun.com/mirror/docker-ce?spm=a2c6h.13651102.0.0.2cd51b11kEu5bg)

感觉使用docker运行一些持久化工具更方便所以我装了mysql和redis

### Mysql

正常运行mysql需要先启动一次mysql，然后进去将部分类容拷贝出来

```shell
docker run -d \
-p 3306:3306 \
--name mysql \
-e MYSQL_ROOT_PASSWORD=123  \
mysql:8.0-debian
# 直接拷贝文件到想要的目录下
docker cp mysql:/etc/mysql /usr/local/mysql/conf
```

然后停掉容器，删除容器，新建容器

```shell
docker run --name mysql \
-p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=123456 \
-v /usr/local/mysql/conf:/etc/mysql \
-v /usr/local/mysql/data:/var/lib/mysql \
-d mysql:8.0-debian
```

使用以下命令可以进入docker使用sql语句查询

```shell
docker exec -it mysql mysql -u root -p
```

在做后端开发的时候有时候会遇到`Public Key Retrieval is not allowed`的错误，可以参考[这里](https://blog.51cto.com/lwops/2491968)配置mysql证书

主要命令是：

```shell
openssl genrsa 2048 > ca-key.pem #生成CA私钥
openssl req -new -x509 -nodes -days 99999 -key ca-key.pem -out ca.pem #生成数字证书
openssl req -newkey rsa:2048 -days 99999 -nodes -keyout server-key.pem -out server-req.pem #创建mysql私钥和请求证书
openssl rsa -in server-key.pem -out server-key.pem #私钥转换为RSA格式
openssl x509 -req -in server-req.pem -days 99999 -CA ca.pem -CAkey ca-key.pem -set_serial 01 -out server-cert.pem # 用CA证书生成一个服务端的数字证书
openssl req -newkey rsa:2048 -days 99999 -nodes -keyout client-key.pem -out client-req.pem #创建客户端RSA私钥和证书
openssl rsa -in client-key.pem -out client-key.pem #将私钥转换为RSA格式
openssl x509 -req -in client-req.pem -days 99999 -CA ca.pem -CAkey ca-key.pem -set_serial 01 -out client-cert.pem #用CA证书生成一个客户端证书\
#使用cp命令移动所有证书到 /usr/local/mysql/conf中，对应容器中的/etc/mysql
#修改mysql配置文件(/usr/local/mysql/conf/my.cnf)
#添加这几行
ssl-ca=/etc/mysql/ca.pem
ssl-cert=/etc/mysql/server-cert.pem
ssl-key=/etc/mysql/server-key.pem
```



### Redis

创建`/home/redis/data`目录和`/home/redis/redis.conf`文件

redis.conf:

```properties
# 修改配置
daemonize no  #后台启动(注意这里要改为no，即非后台启动，因为会和docker run -d 冲突)
 
# 关闭保护模式，开启的话，只有本机才可以访问redis
protected-mode no  
 
# 需要注释掉bind
bind 127.0.0.1（bind绑定的是自己机器网卡的ip，如果有多块网卡可以配多个ip，代表允许客户端通过机器的哪些网卡ip去访问，内网一般可以不配置bind，注释掉即可）
 
# 设置登录密码
requirepass 123456
 
# 开启aof持久化
appendonly yes
```

然后运行

```shell
docker run --name redis \
-v /home/redis/data:/data \
-v /home/redis/redis.conf:/etc/redis.conf \
-p 6379:6379 \
-d redis:6.0
```

使用以下命令可以进入Redis查询缓存

```shell
docker exec -it redis redis-cli
```

# 一些工具

## 显示网速

```shell
nload -m
```

# 个性软件的安装

## 网易云音乐

电脑怎么能少了网易云呢？🤪🤪🤪🤪🤪🤪🤪

从[网易云音乐官网](https://d1.music.126.net/dmusic/netease-cloud-music_1.2.1_amd64_ubuntu_20190428.deb)可以下载到.deb格式的安装包,我的是Ubuntu20.04还能用，不顾刚下载的时候打不开(好用，要是能打开就好了🤪)

解决方案 参考自[这里](https://blog.csdn.net/luoweid/article/details/124484949)和其评论区

修改`/opt/netease/netease-cloud-music/netease-cloud-music.bash` 为以下内容即可，不用在下载额外的文件

```properties
#!/bin/sh
#HERE="$(dirname "$(readlink -f "${0}")")"
HERE=/opt/netease/netease-cloud-music
export LD_LIBRARY_PATH="${HERE}"/libs
export QT_PLUGIN_PATH="${HERE}"/plugins 
export QT_QPA_PLATFORM_PLUGIN_PATH="${HERE}"/plugins/platforms
cd /lib/x86_64-linux-gnu/
exec "${HERE}"/netease-cloud-music $@
```

不过这里还是放上附件[libgio-2.0.so.0](./assets/libgio-2.0.so.0) 、[libselinux.so.1](./assets/libselinux.so.1) 、[libselinux.so.1](./assets/libselinux.so.1)

## Utools

从[Utools官网](https://u.tools/)下载安装包安装即可

我下载安装后无法运行，于是我去网易云里面复制了一个so文件到`/opt/uTools/`目录下就可以了

附件[libcrypto.so.1.1](./assets/libcrypto.so.1.1) 

# 问题解决

## 更新

### 无法更新Ubuntu store

参考自[这里](https://zhuanlan.zhihu.com/p/580897179)

1. 更新snap store

   ```shell
   sudo snap refresh snap-store
   ```

2. 如果更新时显示他正在运行，则更具显示的pid强行停止他

   ```shell
   kill pid
   ```

3. 重新更新 snap store

   ```shell
   sudo snap refresh snap-store
   ```

   

## 安装

### 安装报错has install-snap change in progress

参考自[这里](https://blog.csdn.net/u011870280/article/details/80213866)

1. 运行下面命令查看安装状态

   ```
   snap changes
   ```

2. 找到自己要安装软件的ID终止他

   ```shell
   sudo snap abort ID
   ```

   