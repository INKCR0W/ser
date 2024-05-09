---
sidebar_position: 6
---

## **题目：**
请采用 samba 服务，实现资源共享。 <br />1、在 linux3 上创建 user00-user19 等 20 个用户；user00 和 user01 添 <br />加到 manager 组，user02 和 user03 添加到 dev 组。把用户 user00- <br />user03 添加到 samba 用户。 <br />2、配置 linux3 为 samba 服务器,建立共享目录/srv/sharesmb，共享名 <br />与目录名相同。manager 组用户对 sharesmb 共享有读写权限，dev 组 <br />对 sharesmb 共享有只读权限；用户对自己新建的文件有完全权限， <br />对其他用户的文件只有读权限，且不能删除别人的文件。在本机用 <br />smbclient 命令测试。 <br />3、在 linux4 修改/etc/fstab,使用用户 user00 实现自动挂载 linux3 的 <br />sharesmb 共享到/sharesmb。
## 配置步骤：
## 1小题
一、安装smaba和samba-client<br />yum install samba samba-client.x86_64

二、创建3个shell脚本<br />touch 00-01.sh 02-03.sh 04-19.sh
```shell
groupadd manager                          #创建manager组
for username in user{00..01}              #循环创建用户
do
useradd -g manager $username              #将用户加入manager组
echo "Pass-1234"|passwd --stdin $username #写入用户密码
done
```
```shell
groupadd dev                          #创建manager组
for username in user{02..03}              #循环创建用户
do
useradd -g dev $username              #将用户加入manager组
echo "Pass-1234"|passwd --stdin $username #写入用户密码
done

```
```shell
for username in user{04..19}
do
useradd $username
echo "Pass-1234"|passwd --stdin $username
done
```
 或者写成一个脚本也行<br />![截图 2023-07-15 13-19-09.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1689398402797-23e33eec-60c3-4386-b311-9c7deb480b79.png#averageHue=%23040302&clientId=u75a42cba-634f-4&from=drop&id=uaf8d6c04&originHeight=741&originWidth=826&originalType=binary&ratio=1&rotation=0&showTitle=false&size=96924&status=done&style=none&taskId=ua6639e0a-d7f3-40fc-83e3-677a0f4074b&title=)<br />三、添加samba用户<br />smbpasswd -a user00   #添加user00到samba数据库，设置密码：输入两次密码以确认<br />smbpasswd -a user01<br />smbpasswd -a user02<br />smbpasswd -a user03
## 2小题
```
[sharesmb]                 #共享名
comment = smbgx            #共享描述
path = /srv/sharesmb       #共享路径
browseable = Yes           #是否可见
writable = Yes             #是否可写
write list = @manager      #可写的用户/组
read list = @dev,@manager  #可读的用户/组
valid users= @dev,@manager #可访问的用户/组
create mask = 0744         #文件权限
directory mask = 0744      #目录权限
```
mkidr /srv/sharesmb<br />chmod 777 /srv/sharesmb  <br />chmod o+t /srv/sharesmb/   #配置目录的sbit权限<br />登录测试：<br />systemc	enable smb.service --now<br />Windows：访问\\10.1.220.101<br />Linux：smbclient //10.1.220.101/sharesmb -U user03<br />![截图 2023-07-15 13-32-50.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1689399199820-e62dfd99-0c5c-475d-b588-ad3bb9937133.png#averageHue=%23080605&clientId=uca9d5e5d-e054-4&from=drop&id=u2720d0c1&originHeight=253&originWidth=1089&originalType=binary&ratio=1&rotation=0&showTitle=false&size=43146&status=done&style=none&taskId=uf3731a62-a53a-4289-96d5-efee9887f17&title=)写入测试：

## 3小题
```
//10.1.220.101/sharesmb /sharesmb               cifs    username=user00,password=Pass-1234 0 0 
```
重启后使用df -h查看挂载结果<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1713340048988-c33fa006-dd47-42d9-9fc0-c2b007a8665e.png#averageHue=%23110e0c&clientId=ue9dbbbe6-7d6f-4&from=paste&height=170&id=u83eab99d&originHeight=234&originWidth=993&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=28086&status=done&style=none&taskId=uf27156e3-bff9-4677-9065-cfa194f3db0&title=&width=722.1818181818181)
