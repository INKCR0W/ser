---
sidebar_position: 11
---

mysql服务<br />任务描述：请安装mysql服务，建立数据表。<br />（1）配置linux2为mysql服务器，创建数据库用户xiao，在任意机器上对所有数据库有完全权限。<br />（2）创建数据库userdb；在库中创建表userinfo，表结构如下：

| 字段名 | 数据类型 | 主键 | 自增 |
| --- | --- | --- | --- |
| id | int | 是 | 是 |
| name | varchar(10) | 否 | 否 |
| birthday | datetime | 否 | 否 |
| sex | varchar(5) | 否 | 否 |
| password | varchar(200) | 否 | 否 |

（3）在表中插入2条记录，分别为(1,user1，1999-07-01，男)，(2,user2，1999-07-02，女)，password与name相同，password字段用password函数加密。<br />（4）修改表userinfo的结构，在name字段后添加新字段height(数据类型为float)，更新user1和user2的height字段内容为1.61和1.62。<br />（5）新建/var/mysqlbak/userinfo.txt文件，文件内容如下，然后将文件内容导入到userinfo表中，password字段用password函数加密。<br />3,user3,1.63,1999-07-03,女,user3<br />4,user4,1.64,1999-07-04,男,user4<br />5,user5,1.65,1999-07-05,男,user5<br />6,user6,1.66,1999-07-06,女,user6<br />7,user7,1.67,1999-07-07,女,user7<br />8,user8,1.68,1999-07-08,男,user8<br />9,user9,1.69,1999-07-09,女,user9<br />（6）将表userinfo的记录导出，存放到/var/databak/mysql.sql，字段之间用','分隔。<br />（7）每周五凌晨1:00以root用户身份备份数据库userdb到/var/databak/userdb.sql(含创建数据库命令)。

## 1小题
 yum install mysql-server.x86_64 <br />systemctl start mysqld<br /> mysql_secure_installation <br />mysql -u root -p<br />create user 'xiao'@'%' identified by '123456';<br />grant all privileges on *.* to 'xiao'@'%';<br />flush privileges;

## 2小题
create database userdb charset utf8;<br />use userdb;<br />create table userinfo(id int auto_increment,name varchar(10),birthday datetime,sex varchar(5),password varchar(200),primary key(id));<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1690026980947-478adefe-28b0-4221-9fc9-7e0f4ad0ddaf.png#averageHue=%230f0c0a&clientId=u5ed4c0a2-0a89-4&from=paste&height=264&id=u3acbf75d&originHeight=264&originWidth=951&originalType=binary&ratio=1&rotation=0&showTitle=false&size=22017&status=done&style=none&taskId=ubd67e3c8-35b6-45d1-9a52-2970aad8c19&title=&width=951)
## 3小题
**注：mysql8.0版本取消了password函数加密，可选加密方式有md5/SHA等**<br />insert into userinfo(id,name,birthday,sex,password) values ('1','user1','1999-7-1','男',md5('user1'));<br />insert into userinfo(id,name,birthday,sex,password) values ('2','user1','1999-7-1','女',md5('user2'));<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1690027312827-4d48e089-c189-4951-9cd6-a047772ff872.png#averageHue=%23110f0d&clientId=u5ed4c0a2-0a89-4&from=paste&height=203&id=ud543d5b0&originHeight=203&originWidth=1105&originalType=binary&ratio=1&rotation=0&showTitle=false&size=17817&status=done&style=none&taskId=ue371f4d0-ecfd-4eef-a807-2407f87da3d&title=&width=1105)
## 4小题
alter table userinfo add height float after name;<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1690027364557-d2648faa-fe67-4ad3-804c-5707e3b375c2.png#averageHue=%230e0b09&clientId=u5ed4c0a2-0a89-4&from=paste&height=297&id=uf0725bd3&originHeight=297&originWidth=962&originalType=binary&ratio=1&rotation=0&showTitle=false&size=24123&status=done&style=none&taskId=u8eb5662c-6ef5-48f3-bbac-d0a5930393d&title=&width=962)<br />update userinfo set height=1.61 where name="user1";<br />update userinfo set height=1.62 where name="user2";<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1690027464769-c6eb33ab-1ad5-4110-9c10-9cf4407b3510.png#averageHue=%23120f0d&clientId=u5ed4c0a2-0a89-4&from=paste&height=193&id=u8ae5c274&originHeight=193&originWidth=1227&originalType=binary&ratio=1&rotation=0&showTitle=false&size=19351&status=done&style=none&taskId=u819870b5-0306-47b5-8a0d-11d7a05a637&title=&width=1227)
## 5小题
mkdir -p /var/mysqlbak/<br />touch /var/mysqlbak/userinfo.txt<br /> chmod 777 /var/mysqlbak/userinfo.txt
```
3,user3,1.63,1999-07-03,女,user3
4,user4,1.64,1999-07-04,男,user4
5,user5,1.65,1999-07-05,男,user5
6,user6,1.66,1999-07-06,女,user6
7,user7,1.67,1999-07-07,女,user7
8,user8,1.68,1999-07-08,男,user8
9,user9,1.69,1999-07-09,女,user9
```
mysql -u root<br />mysql> use userdb;<br />SET GLOBAL local_infile = true;  #启用或禁用本地数据加载功能，此设置为单次，重启mysql之后失效<br />SHOW GLOBAL VARIABLES LIKE 'local_infile';  #查看local_infile 状态<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1690030278043-5147775e-3bbb-4b39-acac-1a944c705417.png#averageHue=%230d0b0a&clientId=u5ed4c0a2-0a89-4&from=paste&height=178&id=u039bfcd1&originHeight=178&originWidth=693&originalType=binary&ratio=1&rotation=0&showTitle=false&size=10020&status=done&style=none&taskId=u5a3d90f3-89e9-45d0-b64c-17bd29e0cbe&title=&width=693)
导入userinfo内记录并md5加密password字段load data local infile '/var/mysqlbak/userinfo.txt' into table userinfo fields terminated by ',' lines terminated by '\n' (id,name,height,birthday,sex,password) set password=md5(password);<br />mysql> select * from userinfo;<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1690030865886-4b6125f4-fa69-40fd-8fe3-ea9332ebe932.png#averageHue=%23191511&clientId=u5ed4c0a2-0a89-4&from=paste&height=369&id=u5a5b0f8f&originHeight=369&originWidth=1230&originalType=binary&ratio=1&rotation=0&showTitle=false&size=52249&status=done&style=none&taskId=uddae888b-56f8-4b27-9c55-c599bf9f905&title=&width=1230)
## 6小题
mkdir -p /var/databak/<br />chmod 777 /var/databak/
vi /etc/my.cnf.d/mysql-server.cnf 在[mysqld]下添加以下内容<br />secure-file-priv=/var/databak/<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1690031329973-cef19e79-b4dd-4c01-986c-608e4e81839e.png#averageHue=%23100e0c&clientId=u5ed4c0a2-0a89-4&from=paste&height=172&id=ua337a0e8&originHeight=172&originWidth=693&originalType=binary&ratio=1&rotation=0&showTitle=false&size=10477&status=done&style=none&taskId=ud69319ad-49ac-41ea-a5c4-de13978519f&title=&width=693)<br />mysql> select * from  userinfo into outfile '/var/databak/mysql.sql' fields terminated by ',';<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1690031362954-72e97fbc-ab09-498f-a301-b794926a685e.png#averageHue=%2317110c&clientId=u5ed4c0a2-0a89-4&from=paste&height=407&id=u52ebbbd4&originHeight=407&originWidth=1223&originalType=binary&ratio=1&rotation=0&showTitle=false&size=65385&status=done&style=none&taskId=udbac0e86-ca0f-4b11-9f13-26d8c70cbf3&title=&width=1223)
## 7小题
touch userdb.sql<br />chmod 777 userdb.sql
crontab -u root -e  #写入以下内容0 1 * * 5 mysqldump -u root -p123456 userdb > /var/databak/userdb.sql<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1690032639790-8c9f59f0-cf32-42d9-857c-e4d0a89f18a4.png#averageHue=%23161310&clientId=u5ed4c0a2-0a89-4&from=paste&height=54&id=u6ae77455&originHeight=54&originWidth=974&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7232&status=done&style=none&taskId=ua588ad41-3817-4cf8-905f-7902a5c673d&title=&width=974)<br />date -s "2023-7-22 00:59:57"   #设置时间
