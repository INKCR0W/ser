---
sidebar_position: 9
---

## **题目：**
：请采用 iscsi，搭建存储服务。 <br />1、为 linux8 添加 4 块磁盘，每块磁盘大小为 5G，创建 lvm 卷，卷组名称为 vg1，逻辑卷名称为 lv1，容量为全部空间，格式化为 ext4 格式。 <br />2、使用/dev/vg1/lv1 配置为 iSCSI 目标服务器，为 linux9 提供 iSCSI服务。iSCSI 目标端的 wwn 为 iqn.2008-01.lan.skills:server,iSCSI发起端的 wwn 为 iqn.2008-01.lan.skills:client1. <br />3、配置 linux9 为 iSCSI 客户端，实现 discovery chap 和 session chap双向认证，Target 认证用户名为 IncomingUser，密码为 IncomingPass；Initiator 认证用户名为 OutgoingUser，密码为 OutgoingPass。<br />4、修改/etc/rc.d/rc.local 文件开机自动挂载 iscsi 磁盘到/iscsi 目录。
注：发现认证：Discovery CHAP发生在iSCSI会话建立之前，用于控制哪些initiator可以发现target。<br />会话认证：Session CHAP则是在会话建立时进行，用于控制哪些initiator可以与target建立会话。
## 配置步骤：
# 1小题
## 1、添加磁盘、创建卷组、创建逻辑卷、格式化
### Server2中执行
qemu-img create -f qcow2 /cp/b.qcow2 5G<br />qemu-img create -f qcow2 /cp/c.qcow2 5G<br />qemu-img create -f qcow2 /cp/d.qcow2 5G<br />qemu-img create -f qcow2 /cp/e.qcow2 5G

virsh attach-disk linux8 /cp/b.qcow2 vdb --persistent --subdriver=qcow2<br />virsh attach-disk linux8 /cp/c.qcow2 vdc --persistent --subdriver=qcow2<br />virsh attach-disk linux8 /cp/d.qcow2 vdd --persistent --subdriver=qcow2<br />virsh attach-disk linux8 /cp/e.qcow2 vde --persistent --subdriver=qcow2

### linux8中执行
fdisk -l或lsblk查看磁盘信息<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1682829630957-8e5bbe9f-16d6-404d-be6c-df7eaaa164c0.png#averageHue=%23040200&clientId=u8c9a9024-477c-4&from=paste&height=643&id=rrdQK&originHeight=643&originWidth=813&originalType=binary&ratio=1&rotation=0&showTitle=false&size=74153&status=done&style=none&taskId=u40b99ecc-9b4f-4c82-b3c8-67ab0293beb&title=&width=813)<br />vgcreate vg1 /dev/vdb /dev/vdc /dev/vdd /dev/vde   #将所有相关物理卷加入卷组 查看结果：vgdisplay<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1682829871138-702f4856-d4f0-4e50-8de0-aac2a780e50d.png#averageHue=%230a0600&clientId=u8c9a9024-477c-4&from=paste&height=121&id=C6UrJ&originHeight=121&originWidth=929&originalType=binary&ratio=1&rotation=0&showTitle=false&size=15170&status=done&style=none&taskId=uc15b9dc0-0f9f-4c64-ac0e-42bf3e6eedc&title=&width=929)<br />lvcreate -l +100%free vg1 -n lv1 #划分全部空间到逻辑卷，名称为lv1   查看结果：lvdisplay<br />mkfs.ext4 /dev/vg1/lv1 #格式化逻辑卷为ext4格式
## 2、配置iscsi目标端
```
[root@linux8 ~]# targetcli
/backstores/block create vg1 /dev/vg1/lv1 #指定要发布共享的磁盘，或者说创建块设备
/iscsi/ create wwn=iqn.2023-08.lan.skills:server #创建iSCSI目标
/iscsi/iqn.2023-08.lan.skills:server/tpg1/luns create /backstores/block/vg1 #配置luns
/iscsi/iqn.2023-08.lan.skills:server/tpg1/acls create iqn.2023-08.lan.skills:client #配置acls，指定发起端的iqn

/iscsi/iqn.2023-08.lan.skills:server/tpg1/portals/ delete 0.0.0.0 3260 #删除默认的全部侦听
/iscsi/iqn.2023-08.lan.skills:server/tpg1/portals/ create 10.4.220.108 3260 #指定侦听的IP

/iscsi set discovery_auth enable=1 userid=IncomingUser password=IncomingPass mutual_userid=OutgoingUser mutual_password=OutgoingPass  #启用认证发现功能，配置发现认证和会话认证的用户名和密码
/iscsi/iqn.2023-08.lan.skills:server/tpg1/acls/iqn.2023-08.lan.skills:client/ set auth userid=IncomingUser password=IncomingPass mutual_userid=OutgoingUser mutual_password=OutgoingPass #配置发起端发现认证和会话认证的用户名和密码
注：userid是设置iSCSI客户端连接到iSCSI目标时需要提供的用户名和密码
    mutual是设置iSCSI目标在与客户端建立会话时需要提供的用户名和密码

/iscsi/ get discovery_auth #查看discovery_auth配置结果
saveconfig 									#保存配置
exit 												#退出targetcli模式
```
# 2小题
## 1、配置iscsi发起端---linux9中执行
yum install iscsi* -y<br />systemctl restart iscsid<br />cd /etc/iscsi/
vi initiatorname.iscsi #配置发起端名称InitiatorName=iqn.2023-08.lan.skills:client
vi iscsi.conf #配置CHAP认证密钥基本逻辑：I连T，I提供T端预设的用户名和密码，  T响应I，I提供T端预设的I的用户名和密码<br />node.session.auth.authmethod = CHAP #允许会话认证<br />发起端（Initiator）连接到目标端（Target）时使用的用户名和密码<br />node.session.auth.username = IncomingUser<br />node.session.auth.password = IncomingPass

目标端（Target）在响应发起端（Initiator）的连接请求时使用的用户名和密码<br />node.session.auth.username_in = OutgoingUser<br />node.session.auth.password_in = OutgoingPass

discovery.sendtargets.auth.authmethod = CHAP #允许发现认证<br />发起端（Initiator）在寻找目标端（Target）时使用的用户名和密码<br />discovery.sendtargets.auth.username = IncomingUser<br />discovery.sendtargets.auth.password = IncomingPass<br />目标端（Target）在响应发起端（Initiator）的发现请求时使用的用户名和密码<br />discovery.sendtargets.auth.username_in = OutgoingUser<br />discovery.sendtargets.auth.password_in = OutgoingPass

systemctl restart iscsid #重启iscsi服务让刚刚的配置生效<br />iscsiadm -m discovery -t sendtargets -p 10.1.220.108 #搜索存储<br />iscsiadm -m node -T iqn.2023-08.lan.skills:server -p 10.1.220.108 --login #连接这个搜索到的IQN<br />iscsiadm -m session -o show #查看连接情况

使用lsblk查看是否有一个20G的磁盘出现。
## 2、修改rc.load文件
vi /etc/rc.d/rc.localtouch /var/lock/subsys/local<br />**sudo mount /dev/sda   /iscsi**<br />chmod +x /etc/rc.d/rc.local<br />mkdir  /iscsi
## 3、验证
重启linux9，使用df -h命令查看重启后是否自动挂载sda磁盘到iscsi目录
