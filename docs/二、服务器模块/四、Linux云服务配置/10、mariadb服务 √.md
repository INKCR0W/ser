---
sidebar_position: 10
---

## **题目：**
请安装 mariadb 服务，建立数据表。 
1、配置 linux3 为 mariadb 服务器，创建数据库用户 xiao，在任意机器上对所有数据库有完全权限。
**2、**创建数据库 userdb；在库中创建表 userinfo，表结构如下： 
![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1694499387348-dacb2483-9083-4228-b0b6-10edb6992ee8.png#averageHue=%23eae9e8&clientId=uf77fbeec-1a20-4&from=paste&height=239&id=u423055b2&originHeight=329&originWidth=889&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=49493&status=done&style=none&taskId=u2f207a11-a160-4468-9fdb-e6e5f3f689a&title=&width=646.5454545454545)
3、在表中插入 2 条记录，分别为(1,user1,1.61,2000-07-01，M)， (2,user2，1.62,2000-07-02，F)，password 字段与 name 字段相同， password 字段用 md5 函数加密。 
4、新建/var/mariadb/userinfo.txt 文件，文件内容如下，然后将文件内容导入到 userinfo 表中，password 字段用 md5 函数加密。 
3,user3,1.63,2000-07-03,F,user3 
4,user4,1.64,2000-07-04,M,user4 
5,user5,1.65,2000-07-05,M,user5 
6,user6,1.66,2000-07-06,F,user6 
7,user7,1.67,2000-07-07,F,user7 
8,user8,1.68,2000-07-08,M,user8 
9,user9,1.69,2000-07-09,F,user9 
5、将表 userinfo 中的记录导出，并存放到/var/mariadb/userinfo.sql，字段之间用','分隔。 
6、为 root 用户创建计划任务（day 用数字表示），每周五凌晨 1:00 备份数据库 userdb(含创建数据库命令)到/var/mariadb/userdb.sql。 
（为便于测试，手动备份一次。）
## 配置步骤：
## 1小题
### 1.1 安装和基础配置
```
yum install mariadb* -y
systemctl enable mariadb --now 开机自启并现在启动
mysql_secure_installation 设置root密码  输入后按回车
[root@linux5 ~]# mysql -uroot -p登录数据库
```
### 1.2 创建用户并分配权限
```
MariaDB [(none)]> show databases;查看所有库
MariaDB [(none)]> select user from mysql.user; #查看数据库用户
MariaDB [(none)]> grant all privileges on *.* to xiao@10.1.220.103 identified by '123456'; #授权并创建用户
MariaDB [(none)]> show grants for 'xiao'@'10.1.220.103';   #查看用户权限
```
注：xiao@本地可以在本机上登录，xiao@IP地址，无法在本地登录mariadb
## 2小题
### 2.1创建数据库和创建表
```
create database userdb; #创建数据库
use userdb; 切换到userdb数据库
create table userinfo(id int auto_increment,name varchar(10),height float,birthday datetime,sex varchar(5),password varchar(200) not null,primary key(id));
创建表
```
## 3小题
```
insert into userinfo(id,name,birthday,sex,password) values ('1','user1','2000-7-1','M',md5('user1'));
insert into userinfo(id,name,birthday,sex,password) values ('2','user2','2000-7-1','F',md5('user2'));
```
扩展命令：desc userinfo; #查看表结构
select * from userinfo; #查看userinfo表记录

password字段password加密应写成
insert into userinfo(id,name,birthday,sex,password) values ('1','user1','2000-7-1','M',password('user1'));
![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714270398061-ed719429-61c6-40b2-a762-c549880d01ac.png#averageHue=%23110f0c&clientId=ud110c0e1-36a1-4&from=paste&height=138&id=u50e1d5c1&originHeight=138&originWidth=1463&originalType=binary&ratio=1&rotation=0&showTitle=false&size=15713&status=done&style=none&taskId=u4144a5ca-587a-42dc-b3f3-19374cdf1f9&title=&width=1463)
## 4小题
### 4.1 创建文件路径并复制内容到该文件中
![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714270111775-04bd990a-75ef-4d4d-bbbd-88bce76e97e6.png#averageHue=%23100b06&clientId=ud110c0e1-36a1-4&from=paste&height=201&id=ud0109340&originHeight=201&originWidth=1025&originalType=binary&ratio=1&rotation=0&showTitle=false&size=22188&status=done&style=none&taskId=u35ff57d1-29ea-4449-9ffc-9e5bf6c4808&title=&width=1025)
### 4.1 在mariadb中输入以下命令导入
 load data local infile '/var/mariadb/userinfo.txt' into table userinfo fields terminated by ',' lines terminated by '\n' (id,name,height,birthday,sex,password) set password=md5(password);
 #导入txt文件中的表记录到userinfo表的id,name,height等字段中
![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714270367079-e2579df2-7010-4c41-af19-ec71833f25c4.png#averageHue=%2316120f&clientId=ud110c0e1-36a1-4&from=paste&height=373&id=u259b22c3&originHeight=373&originWidth=1394&originalType=binary&ratio=1&rotation=0&showTitle=false&size=51954&status=done&style=none&taskId=ue4fa5917-9e12-43e8-a773-124112c67d2&title=&width=1394)
## 5小题
```
chmod 777 /var/mariadb #授予权限

vi /etc/selinux/config #建议永久关闭selinux

进入mariadb输入以下命令导出以逗号隔开的记录
select * from userinfo into outfile '/var/mariadb/userinfo.sql' fields terminated by ',';
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714270455893-3634c8a2-dd93-4cf2-9fc3-bb9611f21773.png#averageHue=%231b140e&clientId=ud110c0e1-36a1-4&from=paste&height=247&id=ua75dd761&originHeight=247&originWidth=1347&originalType=binary&ratio=1&rotation=0&showTitle=false&size=46884&status=done&style=none&taskId=u95f1ba62-8702-4897-b8cb-4e89ddc32fe&title=&width=1347)
## 6小题
crontab -e    #使用crontab创建任务，写入以下内容：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1695966737822-fd37cf95-be5a-4b54-8d91-e19abf9a0b27.png#averageHue=%23330e28&clientId=u871f5835-3630-4&from=paste&height=19&id=ud43b6534&originHeight=29&originWidth=1070&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=13066&status=done&style=none&taskId=ua2b9d1bb-0af5-4689-8617-7432972a676&title=&width=713.3333333333334)
crontab -l  #查看任务列表
![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1695966711256-539b712b-eb25-4429-bdab-0d2b9a317393.png#averageHue=%23320c26&clientId=u871f5835-3630-4&from=paste&height=43&id=ua3c1dbd8&originHeight=65&originWidth=1063&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=21311&status=done&style=none&taskId=u30ee5f03-3dd9-4970-a3b8-f757010346e&title=&width=708.6666666666666)
下图看说明：
![图片.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1689685468768-75f09288-83e0-45a7-a293-857adb8ce005.png#averageHue=%23292d36&clientId=ucb0079dc-dfc2-4&from=paste&height=182&id=u22ccce82&originHeight=199&originWidth=780&originalType=binary&ratio=1.0909090909090908&rotation=0&showTitle=false&size=20891&status=done&style=none&taskId=u9b71b2a2-8ba7-498e-80d5-1af34b73aa0&title=&width=715)
