# RabbitMQ的CentOS7安装

## 1. 下载与安装

### Erlang/OTP 的下载安装

如果只是使用 RabbitMQ，推荐使用 RabbitMQ 公司维护的 erlang 版本，该版本只保留了与 RabbltMQ 相关的功能依赖。

地址是：[https://github.com/rabbitmq/erlang-rpm](https://github.com/rabbitmq/erlang-rpm)

#### 安装 Erlang 21.x

```yml
# In /etc/yum.repos.d/rabbitmq-erlang.repo
[rabbitmq-erlang]
name=rabbitmq-erlang
baseurl=https://dl.bintray.com/rabbitmq-erlang/rpm/erlang/21/el/7
gpgcheck=1
gpgkey=https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc
repo_gpgcheck=0
enabled=1
```

```bash
# yum 安装
yum install erlang

# 查看安装成功
erl

Erlang/OTP 21 [erts-10.3.5.4] [source] [64-bit] [smp:1:1] [ds:1:1:10] [async-threads:1] [hipe]

Eshell V10.3.5.4  (abort with ^G)
```

### RabbitMQ 的下载与安装

```bash
# 安装环境依赖
yum install build-essential openssl openssl-devel unixODBC unixODBC-devel make gcc gcc-c++ kernel-devel m4 ncurses-devel tk tc xz

# 安装 socat
yum install socat

# 下载安装包
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.17/rabbitmq-server-3.7.17-1.el7.noarch.rpm

# 安装
yum install rabbitmq-server-3.7.17-1.el7.noarch.rpm

# 查看是否已经安装ok
rpm -qa | grep rabbitmq
```

## 2. 运行 RabbitMQ 服务器

默认情况下，在安装 RabbitMQ 服务器软件包时，服务器不会作为守护程序启动。在系统引导时，默认情况下以管理员身份运行时启动守护程序

```bash
chkconfig rabbitmq-server on
```

### 启动和关闭

```bash
systemctl start rabbitmq-server.service    #启动
rabbitmq-server start &                    #启动2

systemctl stop rabbitmq-server.service     #关闭

systemctl status rabbitmq-server.service   #状态
```

### 开启远程访问

```bash
cd /etc/rabbitmq

cp /usr/share/doc/rabbitmq-server-3.7.17/rabbitmq.config.example /etc/rabbitmq/

mv rabbitmq.config.example rabbitmq.config
```

修改 `rabbitmq.config` 文件

```bash
vi /etc/rabbitmq/rabbitmq.config

# %% {loopback_users, []},
# 修改成如下
{loopback_users, []}
```

## 3. 插件安装

### 查看插件列表

```bash
cd /sbin/

./rabbitmq-plugins list
```

### 安装管理界面插件

这个命令的作用是安装 RabbitMq 的一个管理插件，这样，我们就可以通过在浏览器访问 `http://ip:15672` 时，进入一个管理界面。

```bash
rabbitmq-plugins enable rabbitmq_management
```

## 4. 配置文件

```bash
vi /usr/lib/rabbitmq/lib/rabbitmq_server-3.7.17/ebin/rabbit.app

# 开启远程访问方式二
# {loopback_users, [<<"guest">>]}, 修改成如下
{loopback_users, [guest]},
```

## 命令

有4种命令

```bash
# rabbitmq +tab键

rabbitmqctl     rabbitmq-diagnostics     rabbitmq-plugins     rabbitmq-server
```

### rabbitmqctl 命令操作

```bash
rabbitmqctl stop_app    关闭应用
rabbitmqctl start_app   启动应用
rabbitmqctl status      节点状态
rabbitmqctl add_user    添加用户
rabbitmqctl list_user   列出所有用户
rabbitmqctl delete_user 删除用户
rabbitmqctl clear_permissions -p vhostpath [username]   清除用户权限
rabbitmqctl set_permissions -p vhostpath [username] [permissions]  设置用户权限
rabbitmqctl change_password [username] [newpassword]    修改密码
rabbitmqctl list_user_permissions [username]            列出用户权限

rabbitmqctl add_vhost vhostpath     创建虚拟主机
rabbitmqctl list_vhosts             列出所有虚拟主机
rabbitmqctl delete_vhost vhostpath  删除虚拟主机
rabbitmqctl list_permissions -p vhostpath   列出虚拟主机上所有权限

rabbitmqctl list_queues     查看所有队列信息
rabbitmqctl -p vhostpath purge_queue blue   清楚队列里的消息
```

增加用户 admin 和密码 admin

```bash
# In /sbin/
./rabbitmqctl add_user admin admin

# 设置权限
./rabbitmqctl set_user_tags admin administraotr

# 查看用户列表
./rabbitmqctl list_users
```

### rabbitmqctl 高级操作

```bash
rabbitmqctl reset   移除所有数据，要在 rabbitmqctl stop_app 之后操作
rabbitmqctl join_cluster <clusternode> [--ram/--dist]   组成集群命令
rabbitmqctl cluster_status      查看集群命令
rabbitmqctl change_cluster_node_type disc|ram 修改集群节点的存储形式
rabbitmqctl forget_cluster_node [--offline]     忘记节点（摘除节点）
rabbitmqctl rename_cluster_node oldnode1 newnode1 [oldnode2] [newnode2] ...  (修改节点名称)
```




















