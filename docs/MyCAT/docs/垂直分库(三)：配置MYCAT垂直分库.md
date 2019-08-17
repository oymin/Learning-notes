# 垂直分库(三)：配置 MYCAT 垂直分库

- 使用 schema.xml 配置逻辑库

- 使用 server.xml 配置系统变量及用户权限

- 由于没有用到水平分片所以不需要配置 rule.xml


 create user im_mycat@'192.168.75.%' identified by '123456';

 grant select,insert,update,delete on *.* to im_mycat@'192.168.75.%';

 mysql -uapp_koax -p123456 -h192.168.75.128 -P8066


//grant all privileges on *.* to 'root'@'%' identified by 'jenkins@123' with grant option;


## 清除多余数据

stop slave;

reset slave all;