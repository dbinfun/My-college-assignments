# Linux常用命令

目录:



[TOC]



## 防火墙

### 开启与关闭

CentOS

```shell
systemctl start firewalld.service #开启
systemctl unmask firewalld.service #开启失败可能需要先解锁服务
systemctl stop firewalld.service #关闭
firewall-cmd --reload #重启
systemctl disable firewalld.service #禁用防火墙
service network restart #重启网卡
```

Ubuntu

```shell
sudo ufw status #查看防火墙状态
sudo ufw status verbose #更多详细情况
sudo ufw enable #开启防火墙
sudo ufw disable #关闭防火墙
sudo ufw version #查看防火前版本
sudo ufw default allow #默认允许外部访问本机
sudo ufw default deny #拒绝外部访问本机
sudo ufw allow 53 #允许外部访问53端口
sudo ufw deny 53 #拒绝外部访问53端口
sudo ufw allow from 192.168.0.1 #允许某个IP地址访问本机所有端口
sudo ufw status numbered # 输出防火墙规则包括其id(id永远从1开始)
sudo ufw delete 1 #删除id为1的规则
sudo ufw delete allow 8080 #删除8080端口规则
sudo ufw reset #重置防火墙
```

### 查看防火墙

```shell
systemctl status firewalld.service #防火墙状态
firewall-cmd --list-all #查看防火墙开放端口
```

### <a id='防火墙端口开放与关闭'>防火墙端口开放与关闭</a>

```shell
firewall-cmd --permanent --add-port=3306/tcp #开启3306端口,重启后生效
firewall-cmd --reload #重启
firewall-cmd --remove-port=3306/udp --permanent #关闭端口
```

## 进程

### 后台运行脚本

```sh
nohup test.sh > nohup.out  2>&1 &
```

`>nohup.out` 指的是输出到`nohup.out` 也可输出到 `/dev/null`  /dev/null 永远都是空的。

若将`>nohup.out` 换成 `>>nohup.out` 则是追加模式,不会覆盖原内容。

### 通过名称查询进程

```sh
ps -aux | grep 名称 |grep -v grep
```

`grep -v grep` 是排除当前命令

### 结束进程

```sh
kill -9 process_id
```

以上命令就可以构成简单的脚本

```sh
kill_pid=$(ps -aux | grep openfire |grep -v grep | awk '{print $2}') #查询pid
kill -9 kill_pid #杀掉进程
```

## 端口

### 端口的开放与关闭

见[端口开放与关闭](#防火墙端口开放与关闭)

### 查看端口占用

`lsof`

```shell
lsof -i:端口号
```

`netstat`

**netstat -tunlp** 用于显示 tcp，udp 的端口和进程等相关情况。

netstat 查看端口占用语法格式：

```shell
netstat -tunlp | grep 端口号
```

- -t (tcp) 仅显示tcp相关选项
- -u (udp)仅显示udp相关选项
- -n 拒绝显示别名，能显示数字的全部转化为数字
- -l 仅列出在Listen(监听)的服务状态
- -p 显示建立相关链接的程序名

## SSH的使用

### SSH的安装

使用`rmp`安装

```shell
apt-get install ssh
```

**或者**

使用`yum`安装

```shell
yum -y install openssh
```

### 开启SSH服务

```shell
service sshd start
```

**或者**

```shell
systemctl restart sshd.service
```

### ssh配置文件的修改

```shell
vi /etc/ssh/sshd.config
#or
vim /etc/ssh/sshd.config
```

其中：

- `PermitRootLogin` 允许Root用户登录
- `PubkeyAuthentication` 允许公钥登录
- `PasswordAuthentication` 口令(密码)登录
- `RSAAuthentication` RSA认证

添加公钥

```
vim ~/.ssh/authorized_keys #直接复制公钥追加到authorized_keys中.
systemctl restart sshd.service #可以重启一下服务
```

可以使用git-bash生成密钥对

```shell
git config --global  --list #是否配置用户名和邮箱,若没有执行下面两条命令
git config --global  user.name "这里换上你的用户名" #若第一条命令输出了用户名等则跳过
git config --global user.email "这里换上你的邮箱" #若第一条命令输出了用户名等则跳过
~/.ssh #检查是否生成ssh文件夹,执行命令后切换到.ssh文件夹下。如果没有则自行创建.ssh
ssh-keygen -t rsa -c "这里换上你的邮箱" #运行后根据提示创建密钥
cd ~/.ssh #查看生成的密钥,id_rsa是私钥(打死一不要泄露),id_rsa.pub是公钥
```

### SSH连接服务器

使用账号密码连接

```shell
ssh  root@192.168.1.1  -p 22# 格式为 用户名@ip地址 -p 端口号
```

使用密钥对登录

```shell
ssh  root@192.168.1.1  -p 22 -i "私钥文件的位置" 
```

其中`-i`为指定私钥文件路径,不指定则采用默认位置

使用ssh端口映射

`将本地1000端口映射到192.168.1.2的1001端口`可以同时映射多个(多了`-L`)

```shell
ssh -L 1000:127.0.0.1:1001 root@192.168.1.2 -p 22
```

## 其他应用

### Java

#### 环境配置

```shell
vim /etc/profile #编辑配置文件
```

添加或者修改配置文件

<font color='blue'>JAVA_HOME</font>:jdk目录

```shell
JAVA_HOME=/usr/local/java/jdk1.8.0_221
CLASSPATH=%JAVA_HOME%/lib:%JAVA_HOME%/jre/lib
PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin
export PATH CLASSPATH JAVA_HOME
```

重载配置文件

```shell
source /etc/profile
```

### 启动和关闭jar包运行

启动jar包 退出后依旧运行

```shell
nohup java -jar **.jar >log.txt &
```

查看占用端口 http是80 https是443

```shell
netstat -lnp|grep 443
```

根据进程号关闭进程

```shell
kill -9 id
```

### Mysql

#### 连接数据库

```shell
mysql -h 域名 -P 端口 -u 用户名 -p
```

### Redis

#### 启动与关闭

```sh
redis-server redis.conf #根据redis.conf启动
redis-cli -h 127.0.0.1 -p 6379 shutdown #关闭redis,127.0.0.1，6379是对应ip和端口
```
