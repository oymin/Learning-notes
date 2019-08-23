# CentOS7安装MySQL5.7

## 1. 卸载

安装前检测是否安装过 MySQL，可以选择是否进行卸载

#### 1.1 yum 方式

```bash
#查看yum是否安装过mysql
yum list installed mysql*
```

如或显示了列表，说明系统中有 MySQL：

![image](../../../images/CentOS/01.png)

yum 根据列表上的名字卸载：

```bash
yum remove mysql-community-client mysql-community-common mysql-community-libs mysql-community-libs-compat mysql-community-server mysql57-community-release

rm -rf /var/lib/mysql  
rm /etc/my.cnf
```

#### 1.2 rpm 方式

rpm 查看安装

```bash
rpm -qa | grep -i mysql
```

rpm 卸载：

```bash
#普通删除模式
rpm -e mysql

#强力删除模式，使用上面命令删除提示有依赖的其它文件，则用该命令进行强力删除
rpm -e --nodeps mysql
```

进行卸载

```bash
rpm -e mysql57-community-release-el7-9.noarch
省略....
rpm -e mysql-community-client-5.7.17-1.el7.x86_64

cd /var/lib/  
rm -rf mysql/
```

#### 1.3 清除卸载残留项

```bash
#查询
whereis mysql

#结果
mysql: /usr/bin/mysql /usr/lib64/mysql /usr/local/mysql /usr/share/mysql /usr/share/man/man1/mysql.1.gz

#删除上面的文件夹
rm -rf /usr/bin/mysql
....
```

#### 1.4 删除配置

```bash
rm –rf /usr/my.cnf
rm -rf /root/.mysql_sercret
```

剩余配置检查，列表显示有则删除

```bash
chkconfig --list | grep -i mysql
chkconfig --del mysqld
```

---

## 2. 安装MySQL

#### 2.1 获取 MySQL yum 源

进入 mysql 官网获取 RPM 包下载地址：[https://dev.mysql.com/downloads/repo/yum/](https://dev.mysql.com/downloads/repo/yum/)

或者使用 `wget` 命令获取

```h
wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```

#### 2.2 安装软件源

```css
sudo rpm -Uvh mysql57-community-release-el7-11.noarch.rpm
```

#### 2.3 安装 MySQL 服务端

```css
yum install -y mysql-community-server
```

#### 2.4 启动 MySQL 服务

```css
systemctl start mysqld
```

## 3. 登录 MySQL

#### 3.1 获取 MySQL 临时密码

MySQL5.7 为 root 用户随机生成了一个密码，在 error log 中，关于 error log 的位置，如果安装的是 RPM 包，则默认是 `/var/log/mysqld.log`

```css
grep 'temporary password' /var/log/mysqld.log
```

结果中的默认密码是： `i?AfoKQrH9te`

```bash
2019-08-15T11:06:29.524377Z 1 [Note] A temporary password is generated for root@localhost: i?AfoKQrH9te
```

登录 MySQL

```css
mysql -u root -p i?AfoKQrH9te
```

#### 3.2 修改登录密码

MySQL 5.7默认密码策略要求密码必须是大小写字母数字特殊字母的组合，至少8位

可以通过修改两个参数，来改变 MySQL 的密码判断：

```css
set global validate_password_policy=0;
```

```css
set global validate_password_length=1;
```

执行修改密码：

```css
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
```

#### 3.3 授权其它机器登录

MySQL 默认不允许远程登录，执行以下命令允许远程登录

```css
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '自己的密码' WITH GRANT OPTION;

FLUSH  PRIVILEGES;
```

> **注意：**  
> 如果防火墙没有开放 3306 端口，需要设置防火墙开放 3306 端口

#### 3.4 配置默认编码为 utf8

```css
character_set_server=utf8

init_connect='SET NAMES utf8'
```

重启后登录查看编码设置

```css
systemctl restart mysqld
```

```css
mysql> show variables like '%character%';

+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.01 sec)
```

#### 3.5 设置开机启动

```css
systemctl enable mysqld

systemctl daemon-reload
```

---

## 4. MySQL 相关

#### MySQL 常用指令

```bash
开机自动启动：  systemctl enable mysqld
开机不自动启动：systemctl disable mysqld
停止服务：      systemctl stop mysqld.service
查询状态：      systemctl status mysqld.service
启动服务：      systemctl start mysqld.service
重启服务：      systemctl restart mysqld.service
```

#### MySQL 安装的目录

Centos 通过 yum 安装( RPM 分发进行安装) MySQL 的几个默认目录如下：

| 目录 | 目录内容 |
| :------------- |:-------------|
| /usr/bin | 客户端程序和脚本 |
| /usr/sbin | mysqld服务器 |
| /var/lib/mysql | 日志文件，数据库文件 |
| /var/log/mysqld.log | 日志文件 |
| /usr/share/mysql | 错误消息和字符集文件 |
| /etc/my.cnf | 配置文件 |






