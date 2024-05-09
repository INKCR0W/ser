---
sidebar_position: 7
---

## **题目：**
请采用 nfs，实现共享资源的安全访问。 <br />1、配置 linux2 为 kdc 服务器，负责 linux3 和 linux4 的验证。 <br />2、在 linux3 上，创建用户，用户名为 xiao，uid=2222，gid=2222，家目录为/home/xiaodir。 <br />3、配置 linux3 为 nfs 服务器，目录/srv/sharenfs 的共享要求为：linux <br />服务器所在网络用户有读写权限，所有用户映射为 xiao，kdc 加密方 <br />式为 krb5p。 <br />4、配置 linux4 为 nfs 客户端，利用 autofs 按需挂载 linux3 上的 <br />/srv/sharenfs 到/sharenfs 目录，挂载成功后在该目录创建 test 目 <br />录。
## 配置步骤：
## 1小题
1、所有主机修改hosts<br />vi /etc/hosts<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1682849330061-1de0ab16-c97b-4fab-b8de-f272d33b963d.png#averageHue=%23080300&clientId=u24dab077-4c46-4&from=paste&height=131&id=u30cd9f54&originHeight=131&originWidth=1095&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11522&status=done&style=none&taskId=ue43ac7e8-3e57-44d2-85f2-aa512858ff7&title=&width=1095)<br />2、软件安装<br />yum install krb5*   #Linux2 kdc服务器执行<br />yum install krb5-workstation*  #linux3 和 linux4 执行

3、定义Kerberos领域名以及它们与DNS域名之间的映射关系，Kerberos是nfs服务里用于认证的协议<br />vi /etc/krb5.conf
修改以下位置[libdefaults] #这段定义全局默认参数<br />    default_realm = SKILLS.LAN  #默认的Kerberos领域名

[realms] #定义具体的Kerberos 领域配置
SKILLS.LAN = {
    kdc = linux2.skills.lan  #指定 KDC 的地址或域名
    admin_server = linux2.skills.lan  #指定管理员服务器的地址或域名
}

[domain_realm] #定义域名到领域的映射<br />.skills.lan = SKILLS.LAN #.skills.lan 域名映射到 SKILLS.LAN 领域<br />skills.lan = SKILLS.LAN #将 skills.lan 域名映射到 SKILLS.LAN 领域

![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1682849564923-749126fd-3a9d-42b9-a131-074cea5ce65f.png#averageHue=%23030200&clientId=u24dab077-4c46-4&from=paste&height=661&id=IZ77L&originHeight=661&originWidth=877&originalType=binary&ratio=1&rotation=0&showTitle=false&size=43281&status=done&style=none&taskId=u98545783-3b27-42a5-a4fa-8960122dc45&title=&width=877)<br />4、授权admin@SKILLS.LAN主体的用户对所有kadmin服务的所有操作的访问权限<br />vi /var/kerberos/krb5kdc/kadm5.acl<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1682849680343-bb9c6131-29b5-4ee1-a4ce-e87cff7ea0be.png#averageHue=%23110e09&clientId=u24dab077-4c46-4&from=paste&height=30&id=u459cc016&originHeight=30&originWidth=373&originalType=binary&ratio=1&rotation=0&showTitle=false&size=2000&status=done&style=none&taskId=ucf22048d-67db-46fa-90fa-da45573dcc5&title=&width=373)<br />*代表所有服务<br />5、初始化kdc数据库<br />kdb5_util create -s <br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1694856524684-c7a4eb61-62d8-4d42-9251-e28c2d4cd3c9.png#averageHue=%23310c26&clientId=ue9fa1546-6a9d-4&from=paste&height=193&id=u47153e66&originHeight=212&originWidth=1033&originalType=binary&ratio=1.100000023841858&rotation=0&showTitle=false&size=76898&status=done&style=none&taskId=u23bbbca7-9ea7-4c74-8cd1-042f24e7fd5&title=&width=939.0908887366622)<br />systemctl start krb5kdc.service kadmin.service --now  启动并开机自启<br />scp /etc/krb5.conf root@10.4.220.103:/etc/krb5.conf<br />scp /etc/krb5.conf root@10.4.220.104:/etc/krb5.conf

6、创建主体、key    #kdc服务器执行<br />kadmin.local #进入到kadmin.local的模式<br />addprinc root/admin #创建主体<br />addprinc -randkey "nfs/linux2.skills.lan" #创建节点的key<br />addprinc -randkey "nfs/linux3.skills.lan" #创建节点的key<br />addprinc -randkey "nfs/linux4.skills.lan" #创建节点的key<br />kadmin.local:  listprincs #查看已创建成功的key<br />ktadd nfs/linux2.skills.lan #下载主服务器的key

7、客户端下载节点key<br />linux3执行<br />kadmin 登录数据库 登不了需要放行88/tcp/udp端口<br />ktadd nfs/linux3.skills.lan  下载节点秘钥

linux4执行<br />kadmin <br />ktadd nfs/linux4.skills.lan

## 2小题
linux3执行<br />groupadd -g 2222 xiao<br />useradd -u 2222 -g xiao -d /home/xiaodir xiao<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1694859471960-4160855c-baf5-4b61-aad5-b64a57a70d33.png#averageHue=%23320c26&clientId=ue9fa1546-6a9d-4&from=paste&height=49&id=udf055ffc&originHeight=54&originWidth=829&originalType=binary&ratio=1.100000023841858&rotation=0&showTitle=false&size=15292&status=done&style=none&taskId=uc0eb9bc2-337f-403e-a8f1-2218055f11a&title=&width=753.6363473017357)
## 3小题
linux3执行<br />1、yum install nfs-utils 安装nfs<br />2、mkdir /srv/sharenfs  创建共享目录<br />3、vi /etc/exports<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1694859788198-fb727c65-2129-4b12-8553-68119c21e01b.png#averageHue=%23310b26&clientId=ue9fa1546-6a9d-4&from=paste&height=49&id=u4dbb5118&originHeight=54&originWidth=829&originalType=binary&ratio=1.100000023841858&rotation=0&showTitle=false&size=18626&status=done&style=none&taskId=u334d3ab7-7cbd-4a6d-bd1d-c4f6450086b&title=&width=753.6363473017357)<br />exportfs -rv #加载配置，使其生效<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1694859803895-035044ba-b5fe-4e0d-857a-a0602184e913.png#averageHue=%23320c26&clientId=ue9fa1546-6a9d-4&from=paste&height=47&id=u149bbe25&originHeight=52&originWidth=702&originalType=binary&ratio=1.100000023841858&rotation=0&showTitle=false&size=16245&status=done&style=none&taskId=ub632354f-a73b-4829-bbb8-c79ca10ba00&title=&width=638.1818043496)

5、systemctl restart rpcbind.service 重启rpc服务<br />systemctl restart nfs-server.service 重启nfs服务

6、showmount -e 10.4.220.103 查看挂载情况<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1694859882082-adbf6184-3cae-4306-8a5c-c7e854442b99.png#averageHue=%23310b25&clientId=ue9fa1546-6a9d-4&from=paste&height=75&id=u2ad18927&originHeight=83&originWidth=763&originalType=binary&ratio=1.100000023841858&rotation=0&showTitle=false&size=22300&status=done&style=none&taskId=u6938f599-fdf1-4982-8ec8-c828eee742c&title=&width=693.6363486022007)
## 4小题
linux4执行<br /> 1、yum install autofs nfs-utils<br />2、vi /etc/auto.master  #auto主配置文件<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1694860373195-a4ed686c-5b65-423e-a3fa-09e36e24e7d6.png#averageHue=%23300b25&clientId=ue9fa1546-6a9d-4&from=paste&height=53&id=u2432a148&originHeight=58&originWidth=1009&originalType=binary&ratio=1.100000023841858&rotation=0&showTitle=false&size=11289&status=done&style=none&taskId=u58ffc76d-a290-45ae-ad3a-99eadb0aa11&title=&width=917.2727073913768)<br />3、vi /etc/auto.mis<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1694860340907-44417184-da47-4a8a-967f-84e7a07fcdaa.png#averageHue=%23310c26&clientId=ue9fa1546-6a9d-4&from=paste&height=47&id=u1f01285b&originHeight=52&originWidth=988&originalType=binary&ratio=1.100000023841858&rotation=0&showTitle=false&size=18812&status=done&style=none&taskId=u005dd8b6-df35-44be-abfb-c22aa2af07e&title=&width=898.181798714252)<br />注意：linux3上需要给sharenfs权限，不然挂载的用户没办法创建文件<br />4、systemctl restart nfs-utils.service autofs.service

5、cd /sharenfs  切换到共享目录<br />6、cd sharenfs  触发挂载动作<br />7、df -h 查看挂载情况<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1694860467100-1bda77d0-0e84-424f-8a4a-4f2ddb06bda0.png#averageHue=%23310b25&clientId=ue9fa1546-6a9d-4&from=paste&height=263&id=u9c7a95c9&originHeight=289&originWidth=1077&originalType=binary&ratio=1.100000023841858&rotation=0&showTitle=false&size=89234&status=done&style=none&taskId=uc4c09a75-e202-452d-b1b6-f70ed8851b7&title=&width=979.0908878696856)
