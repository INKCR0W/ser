---
sidebar_position: 2
---

## **题目：**
创建 DNS 服务器，实现企业域名访问。
1、配置 linux 主机的 IP 地址和主机名称。 
2、所有 linux 主机启用防火墙（kubernetes 服务主机除外），防火墙区域为 public，在防火墙中放行对应服务端口。 
3、所有 linux 主机之间（包含本主机）root 用户实现密钥 ssh 认证，禁用密码认证。 
4、利用 chrony，配置 linux1 为其他 linux 主机提供 NTP 服务。
5、利用 bind，配置 linux1 为主 DNS 服务器，linux2 为备用 DNS 服务器，为所有 linux 主机提供冗余 DNS 正反向解析服务。正向区域文件均/var/named/named.skills ， 反 向 区 域 文 件 均 为 /var/named/named.10。 
6、配置 linux1 为 CA 服务器,为 linux 主机颁发证书。证书颁发机构有效期 10 年，公用名为 linux1.skills.lan。申请并颁发一张供 linux 服务器使用的证书，证书信息：有效期=5 年，公用名=skills.lan，国家=CN，省=Beijing，城市=Beijing，组织=skills，组织单位=system， 使用者可选名称=*.skills.lan 和 skills.lan。将证书skills.crt 和私钥 skills.key 复制到需要证书的 linux 服务器/etc/pki/tls 目 录。浏览器访问 https 网站时，不出现证书警告信息。
## 配置步骤：
### 1小题
修改IP地址vi /etc/NetworkManager/system-connections/ens160.nmconnection #使用vi编辑器修改网卡配置文件

[ipv4]
method=manual  				      #取消自动获取，改为manual，manual为手动的意思
address1=10.4.220.100/24,10.4.220.1  #配置IP地址和网关
dns=10.4.220.101;10.4.220.102  	      #dns配置
nmcli connection reload  ens160 加载配置信息
nmcli connection up ens160 激活配置
修改主机名nmcli g hostname linux1
### 2小题
放行端口firewall-cmd --zone=public --add-port=22/tcp --add-port=53/tcp --add-port=53/udp --add-port=123/udp --add-port=80/tcp --add-port=443/tcp --add-port=21/tcp --add-port=8080/tcp --add-port=2049/tcp --add-port=111/tcp --add-port=111/udp --add-port=139/tcp --add-port=445/tcp --add-port=860/tcp --add-port=3260/tcp --add-port=3306/tcp --add-port=8000/tcp --permanent

firewall-cmd --reload   #重启防火墙
firewall-cmd --list-all #查看放行情况
服务端口说明dns：53/tcp 53/udp
chrony端口：123/UDP
ansible：22/tcp
Apache2服务：80/TCP 443
ftp端口：21/TCP  VSFTP
tomcat端口：80/TCP 443/TCP 8080/TCP
nfs端口：2049/TCP 111/TCP  111/UDP
smaba端口：139/TCP 445/TCP
iscsi端口：860/TCP 3260/tcp
mariadb：3306/tcp
podman：8000/udp
### 3小题
### 4小题

![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1713862969706-54673393-ec0f-4649-83a9-91d8e09aaa6b.png#averageHue=%23340f29&clientId=u1ccbc5f7-73d6-4&from=paste&height=88&id=u4da884be&originHeight=99&originWidth=971&originalType=binary&ratio=1.125&rotation=0&showTitle=false&size=41662&status=done&style=none&taskId=u570d0db3-dce8-4777-ae42-20050dfe32a&title=&width=863.1111111111111)
### 5小题
#### Linux1执行
yum install bind bind-utils   #安装dns服务
vi /etc/named.conf 	#编辑主配置文件
![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1694768644277-b8dc003e-926e-44c2-9a4d-e61ea7537dcd.png#averageHue=%23300a25&clientId=u6d4a5990-51b7-4&from=paste&height=224&id=uadcf115d&originHeight=308&originWidth=987&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=57795&status=done&style=none&taskId=ud42b02b5-b7c3-4754-9731-74931c9771e&title=&width=717.8181818181819)
vi /etc/named.rfc1912.zones #编辑区域文件
![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1694768554468-adcb66c4-a60f-40e3-a246-ff5278c96a6e.png#averageHue=%23300a24&clientId=u6d4a5990-51b7-4&from=paste&height=237&id=u9d22180f&originHeight=326&originWidth=518&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=35361&status=done&style=none&taskId=u3b86f3d1-5f26-4d2a-abf9-d7bb6e49808&title=&width=376.72727272727275)
cd /var/named/  #切换到配置文件目录
cp -p named.localhost named.skills #复制正向区域模板文件到新文件
cp -p named.loopback named.10	#复制反向区域模板到新文件
vi named.skills #编辑新的正向区域文件
![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1694768770130-49c9b868-563c-4359-8b30-50c5619d2ed4.png#averageHue=%23300a24&clientId=u6d4a5990-51b7-4&from=paste&height=316&id=ue50060bc&originHeight=434&originWidth=838&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=48717&status=done&style=none&taskId=u5d9bd0fa-b94a-4eae-aa66-bcbfa92eb3b&title=&width=609.4545454545455)
vi named.10 编辑新的反向区域文件
![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1694768796922-f3abc66a-3d3a-4419-a77b-59ff5ded0cdc.png#averageHue=%23300a24&clientId=u6d4a5990-51b7-4&from=paste&height=337&id=u82999d7e&originHeight=464&originWidth=812&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=55393&status=done&style=none&taskId=ub7a8e171-37db-4958-8774-a2d85d773da&title=&width=590.5454545454545)
systemc	restart named  #重启dns服务
测试正向解析![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1694768911842-b24cfb9c-a0d2-49d0-8dd2-45daddc87dbb.png#averageHue=%23300a25&clientId=u6d4a5990-51b7-4&from=paste&height=239&id=LuMEG&originHeight=329&originWidth=1128&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=63040&status=done&style=none&taskId=ue1bc3287-d8eb-49e9-801d-54868c02c74&title=&width=820.3636363636364)
测试反向解析![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1694769008189-797f400c-6572-477d-b700-d87490c844ac.png#averageHue=%23300a24&clientId=u6d4a5990-51b7-4&from=paste&height=608&id=u775a0d5e&originHeight=836&originWidth=1310&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=112998&status=done&style=none&taskId=u301fa0ef-d101-4896-99da-3257e25e9af&title=&width=952.7272727272727)
#### Linux2执行
yum install bind bind-utils   #安装dns服务
vi /etc/named.conf 	#编辑主配置文件
![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1694769352579-49051a10-e119-4c52-a038-f4aacb240a43.png#averageHue=%23300a25&clientId=u6d4a5990-51b7-4&from=paste&height=215&id=ud16ac350&originHeight=296&originWidth=1178&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=58245&status=done&style=none&taskId=ue5c8f0c4-82a1-4ef8-b1af-485b1402233&title=&width=856.7272727272727)
vi /etc/named.rfc1912.zones #编辑区域文件
![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1713330978138-9ba7ddeb-2647-4063-8ce8-b47e244467ce.png#averageHue=%230a0806&clientId=uac37162c-3352-4&from=paste&height=272&id=u1c569668&originHeight=306&originWidth=761&originalType=binary&ratio=1.125&rotation=0&showTitle=false&size=19694&status=done&style=none&taskId=uaedbe2be-6f9b-4f47-85b8-33f649aa6e8&title=&width=676.4444444444445)systemctl restart named  #重启dns服务
systemctl stop named #关闭linux1的dns服务
测试辅助dns服务器的dns解析
正向测试
![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1694769541094-63e1510b-e589-4281-84f8-49709287d0e0.png#averageHue=%23300a24&clientId=u6d4a5990-51b7-4&from=paste&height=273&id=u4984d80f&originHeight=376&originWidth=1304&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=64286&status=done&style=none&taskId=u34fb00d9-08ed-4e39-8e16-2d0d747f25e&title=&width=948.3636363636364)
反向测试
![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1694769584296-9dd16d7c-48bd-464a-bf6f-a7f531a3f5c2.png#averageHue=%23300a24&clientId=u6d4a5990-51b7-4&from=paste&height=572&id=ude4033bc&originHeight=787&originWidth=1299&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=102697&status=done&style=none&taskId=u27839020-3462-4744-927f-deb756d50a0&title=&width=944.7272727272727)
#### 结论
主dns服务器关闭的情况下，备用dns服务能正常解析
### 6小题
#### 方案一
##### 步骤一、根CA证书配置
```
yum install  openssl-perl
cd /etc/pki/CA/
touch serial
echo 00 >> serial
touch index.txt
```
```
openssl genrsa -out cacert.key 2048
```
```
openssl req -new -out ca.csr -key cacert.key
```
```
openssl x509 -req -days 3650 -in ca.csr -signkey cacert.key -out cacert.crt
```
##### 阶段二、SSL证书配置
```
req_extensions = v3_req  取消注释这一行 167行
# extendedKeyUsage = critical,timeStamping
[ v3_req ]
subjectAltName = @alt_names 在v3字段下添加这一行
# Extensions to add to a certificate request
basicConstraints = CA:TRUE   改为TRUE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
[alt_names]      添加字段
DNS.1=skills.com  添加行
DNS.2=*.skills.com 添加行
```
```
openssl genrsa -out skills.key 2048
```
```
openssl req -new -key skills.key -out skills.csr -config /etc/pki/tls/openssl.cnf -extensions v3_req
```
```
openssl ca -in skills.csr -out skills.crt -cert cacert.crt  -keyfile cacert.key -extensions v3_req -days 1825 -config /etc/pki/tls/openssl.cnf
```
##### 阶段三、CA证书转换
openssl pkcs12 -export -out cacert.pfx -inkey cacert.key -in cacert.crt 
openssl pkcs12 -in cacert.pfx -nodes -out cacert.pem
##### 阶段四、配置信任
for i in {1..9};do scp skills.* cacert.pem 10.4.220.10$i:/etc/pki/tls/ ;done
cat cacert.pem >> /etc/pki/tls/certs/ca-bundle.crt   #每一台有证书的主机执行
#### 方案二
##### 步骤一、根CA证书配置
```
yum install  openssl-perl -y
cd /etc/pki/CA/
touch serial
echo 00 >> serial
touch index.txt
```
```
openssl genrsa -out cakey.pem 2048
cp cakey.pem /etc/pki/CA/private/		#签发客户证书需要把ca私钥放到这个位置
```
```
openssl req -new -x509 -key cakey.pem -days 3650 -out cacert.pem
```
##### 阶段二、SSL证书配置
```
vim file.ext
subjectAltName = @alt_names
[alt_names]
DNS.1=skills.lan
DNS.2=*.skills.lan
```
```
openssl genrsa -out skills.key 2048
```
```
openssl req -new -key skills.key -out skills.csr
```
```
openssl ca -in skills.csr -out skills.crt -days 1825 -extfile file.ext
```
##### **阶段三、配置信任**
for i in {1..9};do scp skills.* cacert.pem 10.4.220.10$i:/etc/pki/tls/ ;done
cat cacert.pem >> /etc/pki/tls/certs/ca-bundle.crt   #每一台有证书的主机执行
扩展key和crt 转为pfx
openssl pkcs12 -export -out skills.pfx -inkey skills.key -in skills.crt 

pfx转为pem
openssl pkcs12 -in skills.pfx -nodes -out skills.pem

从pem证书提取crt和key
openssl x509 -in skills.pem -out apache.crt
openssl rsa -in skills.pem -out apache.key

转jks
yum install javapackages-tools.noarch -y
keytool -importkeystore -srckeystore skills.pfx -destkeystore skills.jks -deststoretype JKS

