# 7、nfs服务 √

## **<font style="color:rgb(0,0,0);">题目：</font>**
<font style="color:rgb(0,0,0);">请采用 nfs，实现共享资源的安全访问。 </font>

<font style="color:rgb(0,0,0);">1、配置 linux2 为 kdc 服务器，负责 linux3 和 linux4 的验证。 </font>

<font style="color:rgb(0,0,0);">2、在 linux3 上，创建用户，用户名为 xiao，uid=2222，gid=2222，家目录为/home/xiaodir。 </font>

<font style="color:rgb(0,0,0);">3、配置 linux3 为 nfs 服务器，目录/srv/sharenfs 的共享要求为：linux </font>

<font style="color:rgb(0,0,0);">服务器所在网络用户有读写权限，所有用户映射为 xiao，kdc 加密方 </font>

<font style="color:rgb(0,0,0);">式为 krb5p。 </font>

<font style="color:rgb(0,0,0);">4、配置 linux4 为 nfs 客户端，利用 autofs 按需挂载 linux3 上的 </font>

<font style="color:rgb(0,0,0);">/srv/sharenfs 到/sharenfs 目录，挂载成功后在该目录创建 test 目 </font>

<font style="color:rgb(0,0,0);">录。</font>

## <font style="color:rgb(0,0,0);">配置步骤：</font>
## 1小题
1、所有主机修改hosts

vi /etc/hosts

![1682849330061-1de0ab16-c97b-4fab-b8de-f272d33b963d.png](./img/BWm6uAHgjeY0ikU2/1682849330061-1de0ab16-c97b-4fab-b8de-f272d33b963d-445191.png)

2、软件安装

yum install krb5*   #Linux2 kdc服务器执行

yum install krb5-workstation*  #linux3 和 linux4 执行



3、<font style="color:rgb(7, 19, 62);">定义Kerberos领域名以及它们与DNS域名之间的映射关系，Kerberos是nfs服务里用于认证的协议</font>

<font style="color:rgb(7, 19, 62);">vi /etc/krb5.conf</font>

<details class="lake-collapse"><summary id="u1ecad70d"><span class="ne-text" style="font-size: 16px">修改以下位置</span></summary><p id="u8fdca089" class="ne-p"><span class="ne-text" style="color: rgb(7, 19, 62); font-size: 16px">[libdefaults] #这段定义全局默认参数</span></p><p id="u9615f421" class="ne-p"><span class="ne-text" style="color: rgb(7, 19, 62); font-size: 16px">    </span><span class="ne-text" style="color: #DF2A3F; font-size: 16px">default_realm = SKILLS.LAN  #</span><span class="ne-text" style="color: rgb(7, 19, 62); font-size: 16px">默认的Kerberos领域名</span></p><p id="ue0c5ee1a" class="ne-p"><span class="ne-text" style="color: #DF2A3F; font-size: 16px"></span></p><p id="u779e1b85" class="ne-p"><span class="ne-text" style="color: rgb(7, 19, 62); font-size: 16px">[realms] #定义具体的Kerberos 领域配置</span></p><p id="ucc93be46" class="ne-p"><span class="ne-text" style="color: #DF2A3F; font-size: 16px">SKILLS.LAN = {</span></p><p id="u0e098b52" class="ne-p"><span class="ne-text" style="color: #DF2A3F; font-size: 16px">     kdc = linux2.skills.lan  #</span><span class="ne-text" style="color: rgb(7, 19, 62); font-size: 16px">指定 KDC 的地址或域名</span></p><p id="u15185ad3" class="ne-p"><span class="ne-text" style="color: #DF2A3F; font-size: 16px">     admin_server = linux2.skills.lan  #</span><span class="ne-text" style="color: rgb(7, 19, 62); font-size: 16px">指定管理员服务器的地址或域名</span></p><p id="u55b9e8d0" class="ne-p"><span class="ne-text" style="color: rgb(7, 19, 62); font-size: 16px"> }</span></p><p id="ubcbf75de" class="ne-p"><span class="ne-text" style="color: rgb(7, 19, 62); font-size: 16px"></span></p><p id="ud0183352" class="ne-p"><span class="ne-text" style="color: rgb(7, 19, 62); font-size: 16px">[domain_realm] #定义域名到领域的映射</span></p><p id="u8e6696e2" class="ne-p"><span class="ne-text" style="color: #DF2A3F; font-size: 16px">.skills.lan = SKILLS.LAN #</span><span class="ne-text" style="color: rgb(7, 19, 62); font-size: 14px">.skills.lan</span><span class="ne-text" style="color: rgb(7, 19, 62); font-size: 16px"> 域名映射到 </span><span class="ne-text" style="color: rgb(7, 19, 62); font-size: 14px">SKILLS.LAN</span><span class="ne-text" style="color: rgb(7, 19, 62); font-size: 16px"> 领域</span></p><p id="uf538a478" class="ne-p"><span class="ne-text" style="color: #DF2A3F; font-size: 16px">skills.lan = SKILLS.LAN #</span><span class="ne-text" style="color: rgb(7, 19, 62); font-size: 16px">将 </span><span class="ne-text" style="color: rgb(7, 19, 62); font-size: 14px">skills.lan</span><span class="ne-text" style="color: rgb(7, 19, 62); font-size: 16px"> 域名映射到 </span><span class="ne-text" style="color: rgb(7, 19, 62); font-size: 14px">SKILLS.LAN</span><span class="ne-text" style="color: rgb(7, 19, 62); font-size: 16px"> 领域</span></p><p id="ub20334e0" class="ne-p"><span class="ne-text" style="color: rgb(7, 19, 62); font-size: 16px"></span></p><p id="u5260d76e" class="ne-p"><img src="https://cdn.nlark.com/yuque/0/2023/png/33622884/1682849564923-749126fd-3a9d-42b9-a131-074cea5ce65f.png" width="877" id="IZ77L" class="ne-image"></p></details>
4、授权admin@SKILLS.LAN主体的用户对所有kadmin服务的所有操作的访问权限

vi /var/kerberos/krb5kdc/kadm5.acl

![1682849680343-bb9c6131-29b5-4ee1-a4ce-e87cff7ea0be.png](./img/BWm6uAHgjeY0ikU2/1682849680343-bb9c6131-29b5-4ee1-a4ce-e87cff7ea0be-971938.png)

*代表所有服务

5、初始化kdc数据库

kdb5_util create -s 

![1694856524684-c7a4eb61-62d8-4d42-9251-e28c2d4cd3c9.png](./img/BWm6uAHgjeY0ikU2/1694856524684-c7a4eb61-62d8-4d42-9251-e28c2d4cd3c9-080986.png)

systemctl start krb5kdc.service kadmin.service --now  启动并开机自启

scp /etc/krb5.conf root@10.4.220.103:/etc/krb5.conf

scp /etc/krb5.conf root@10.4.220.104:/etc/krb5.conf



6、创建主体、key    #kdc服务器执行

<font style="color:black;">kadmin.local #进入到kadmin.local的模式</font>

<font style="color:black;">addprinc root/admin</font><font style="color:black;"> #</font><font style="color:black;">创建主体</font>

<font style="color:black;">addprinc -randkey "nfs/linux2.skills.lan" #创建节点的key</font>

<font style="color:black;">addprinc -randkey "nfs/linux3.skills.lan" #创建节点的key</font>

<font style="color:black;">addprinc -randkey "nfs/linux4.skills.lan" #创建节点的key</font>

<font style="color:black;">kadmin.local:  listprincs #查看已创建成功的key</font>

<font style="color:black;">ktadd nfs/linux2.skills.lan #下载主服务器的key</font>

<font style="color:black;"></font>

<font style="color:black;">7、客户端下载节点key</font>

<font style="color:black;">linux3执行</font>

<font style="color:black;">kadmin 登录数据库 登不了需要放行88/tcp/udp端口</font>

<font style="color:black;">ktadd nfs/linux3.skills.lan  下载节点秘钥</font>

<font style="color:black;"></font>

<font style="color:black;">linux4执行</font>

<font style="color:black;">kadmin </font>

<font style="color:black;">ktadd nfs/linux4.skills.lan</font>



## 2小题
linux3执行

groupadd -g 2222 xiao

useradd -u 2222 -g xiao -d /home/xiaodir xiao

![1694859471960-4160855c-baf5-4b61-aad5-b64a57a70d33.png](./img/BWm6uAHgjeY0ikU2/1694859471960-4160855c-baf5-4b61-aad5-b64a57a70d33-209715.png)

## 3小题
linux3执行

1、yum install nfs-utils 安装nfs

2、mkdir /srv/sharenfs  创建共享目录

3、vi /etc/exports

![1694859788198-fb727c65-2129-4b12-8553-68119c21e01b.png](./img/BWm6uAHgjeY0ikU2/1694859788198-fb727c65-2129-4b12-8553-68119c21e01b-195257.png)

exportfs -rv #加载配置，使其生效

![1694859803895-035044ba-b5fe-4e0d-857a-a0602184e913.png](./img/BWm6uAHgjeY0ikU2/1694859803895-035044ba-b5fe-4e0d-857a-a0602184e913-971334.png)



5、systemctl restart rpcbind.service 重启rpc服务

systemctl restart nfs-server.service 重启nfs服务



6、showmount -e 10.4.220.103 查看挂载情况

![1694859882082-adbf6184-3cae-4306-8a5c-c7e854442b99.png](./img/BWm6uAHgjeY0ikU2/1694859882082-adbf6184-3cae-4306-8a5c-c7e854442b99-788519.png)

## 4小题
linux4执行

 1、yum install autofs nfs-utils

2、vi /etc/auto.master  #auto主配置文件

![1694860373195-a4ed686c-5b65-423e-a3fa-09e36e24e7d6.png](./img/BWm6uAHgjeY0ikU2/1694860373195-a4ed686c-5b65-423e-a3fa-09e36e24e7d6-337130.png)

3、vi /etc/auto.mis

![1694860340907-44417184-da47-4a8a-967f-84e7a07fcdaa.png](./img/BWm6uAHgjeY0ikU2/1694860340907-44417184-da47-4a8a-967f-84e7a07fcdaa-736420.png)

注意：linux3上需要给sharenfs权限，不然挂载的用户没办法创建文件

4、systemctl restart nfs-utils.service autofs.service



5、cd /sharenfs  切换到共享目录

6、cd sharenfs  触发挂载动作

7、df -h 查看挂载情况

![1694860467100-1bda77d0-0e84-424f-8a4a-4f2ddb06bda0.png](./img/BWm6uAHgjeY0ikU2/1694860467100-1bda77d0-0e84-424f-8a4a-4f2ddb06bda0-826526.png)



> 更新: 2024-04-24 20:30:38  
> 原文: <https://www.yuque.com/gengmouren-1f9qn/whktvz/lgunni6fgnmnctah>