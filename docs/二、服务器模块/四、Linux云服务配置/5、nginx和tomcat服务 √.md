---
sidebar_position: 5
---

## **题目：**
1、配置 linux2 为 nginx 服务器，默认文档 index.html 的内容为 “HelloNginx”；仅允许使用域名访问，http 访问自动跳转到 https。 <br />2、利用 nginx 反向代理，实现 linux3 和 linux4 的 tomcat 负载均衡， 通过https://tomcat.skills.lan 加密访问 Tomcat，http 访问通过 301 自动跳转到 https。 <br />3、配置 linux3 和 linux4 为 tomcat 服务器，网站默认首页内容分别为 <br />“tomcatA”和“tomcatB”，采用修改配置文件端口形式，仅使用域名 <br />访问 80 端口 http 和 443 端口 https。
## 配置步骤：
## 准备工作：
1、关闭防火墙<br />2、关闭selinux<br />3、dns服务，加入tomcat与Linux2的解析记录

做题顺序：3 - 1 - 2
## 3小题 
yum install java-17-openjdk* tomcat* vim -y
### 3.1 配置Java环境变量
 vi /etc/profile 编辑该文件配置JAVA环境变量<br />在文件尾部添加：<br />export JAVA_HOME=/usr/lib/jvm/java-17<br />export PATH=$JAVA_HOME/bin:$PATH<br />source /etc/profile  #让该文件生效<br />java -version #查看Java版本是否是环境变量中的版本
### 3.2 将tomcat加入systemctl进行管理
vim /usr/lib/systemd/system/tomcat.service<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1690043485393-0c46fe7c-1821-4905-b476-53350219a6f9.png#averageHue=%23060202&clientId=uc9350964-e67f-4&from=paste&height=82&id=ub5af0ae4&originHeight=82&originWidth=1207&originalType=binary&ratio=1&rotation=0&showTitle=false&size=10485&status=done&style=none&taskId=u522cc48b-3310-4ad5-80cf-c84dc54ba09&title=&width=1207)
### 3.2 证书服务器上转换证书
openssl pkcs12 -export -in skills.crt -inkey skills.key -out skills.pfx #转换原有的crt证书为pfx证书，记得要设证书密码，简单的密码就行<br />yum install javapackages-tools.noarch #安装转jks格式的工具 <br />keytool -importkeystore -srckeystore skills.pfx -destkeystore skills.jks -deststoretype JKS #转换pfx证书成jks证书<br />scp skills.jks root@10.1.220.103:/etc/ssl/  #将证书发送到linux3的ssl目录里<br />scp skills.jks root@10.1.220.104:/etc/ssl/  #将证书发送到linux4的ssl目录里<br />scp skills.crt skills.key root@10.1.220.102:/etc/ssl/
### 3.6 配置Tomcat
linux3和linux4配置一样,注意调换IP地址和域名就行<br />vim /etc/tomcat/server.xml <br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1696060479979-1741e867-882c-45c0-9d60-698c3afeb4d9.png#averageHue=%23310a25&clientId=u23f63f70-bd59-4&from=paste&height=544&id=u43a3ac3a&originHeight=816&originWidth=1449&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=166825&status=done&style=none&taskId=u98e0499c-6407-46c1-afe4-69eed573139&title=&width=966)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1696060425513-2dadeb11-a10f-4c0a-ba48-aaa721b7029a.png#averageHue=%23310b25&clientId=u23f63f70-bd59-4&from=paste&height=157&id=u42bb2653&originHeight=235&originWidth=1100&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=53484&status=done&style=none&taskId=u6ebfd364-bde2-4769-aaf4-db32765079b&title=&width=733.3333333333334)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1696060391720-cd04de5d-3834-4fe6-9392-d2ba28828663.png#averageHue=%23320b26&clientId=u23f63f70-bd59-4&from=paste&height=691&id=u3b5e4661&originHeight=1036&originWidth=1455&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=230605&status=done&style=none&taskId=u39e1d41f-54ab-4b74-b890-68f00a4ee67&title=&width=970)<br /> echo "TomcatA" > /usr/share/tomcat/webapps/ROOT/index.jsp #修改写入内容TomcatA<br />systemctl restart tomcat.service<br />检查80和433端口在Tomcat中是否正常启动<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1690043806316-59c110cb-e04c-48bb-830d-80070587ae80.png#averageHue=%230f0c0a&clientId=uc9350964-e67f-4&from=paste&height=134&id=u850d6a68&originHeight=134&originWidth=1421&originalType=binary&ratio=1&rotation=0&showTitle=false&size=22553&status=done&style=none&taskId=uf5115854-1805-43aa-b5c3-16227cb590c&title=&width=1421)<br />测试1：http和https下域名的访问情况<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1696060573675-3523f18e-768f-47d5-8297-ffb0c1b3e7a1.png#averageHue=%23e6f4e9&clientId=u23f63f70-bd59-4&from=paste&height=71&id=ua4818720&originHeight=107&originWidth=514&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=18558&status=done&style=none&taskId=ufac28278-4433-44d2-a010-90016680692&title=&width=342.6666666666667)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1696060537518-5322b5c2-cf87-487a-81fd-4d33c2375e55.png#averageHue=%23e6f4e9&clientId=u23f63f70-bd59-4&from=paste&height=79&id=u22ad1c9f&originHeight=119&originWidth=538&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=19898&status=done&style=none&taskId=u2dd23a46-e439-4cbf-b9af-5a77b772ec7&title=&width=358.6666666666667)<br />上图所示：通过域名正常访问<br />测试2：通过IP方式的访问情况<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1696060608975-7955cdf8-0681-4ef6-a467-7b55db8306d8.png#averageHue=%23f7f9e0&clientId=u23f63f70-bd59-4&from=paste&height=154&id=u1ff9b61a&originHeight=231&originWidth=504&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=28145&status=done&style=none&taskId=ucbbb412e-0b55-4ecb-8a39-0711d357cc6&title=&width=336)<br />上图所示：通过IP无法访问
## 1小题-2小题
### 1、安装
yum install vim nginx -y #安装vim编辑器和nginx服务

### 2、修改selinux：输入以下配置
setsebool -P httpd_can_network_connect 1或直接关闭selinux

### 3、让主配置文件的server字段不生效
vi /etc/nginx/nginx.conf #编辑该文件，注释掉下图内容<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1690044981113-0645f110-a982-4316-91e2-6fb24e876b76.png#averageHue=%23020101&clientId=uc9350964-e67f-4&from=paste&height=888&id=u3a067b04&originHeight=888&originWidth=1446&originalType=binary&ratio=1&rotation=0&showTitle=false&size=89509&status=done&style=none&taskId=u34dcf13a-7efe-4c2c-a985-815d944f393&title=&width=1446)
### 4、配置负载均衡和自动跳转
```
server {
listen 80 default_server;     #用default_server指令禁止IP访问
listen 443 default_server;    #用default_server指令禁止IP访问
server_name _或10.4.220.102;	#
return 403;	#返回403禁止访问
ssl_certificate /etc/ssl/skills.crt;
ssl_certificate_key /etc/ssl/skills.key;
}

server {
listen 80;
server_name linux2.skills.lan;

rewrite ^(.*) https://$server_name$1 permanent;   #通过80端口访问时301跳转到https
}

server {
listen 443 ssl http2;  #通过443端口访问时
server_name linux2.skills.lan;
root /var/nginx;	#访问到的目录
ssl_certificate /etc/ssl/skills.crt;	#绑定的证书
ssl_certificate_key /etc/ssl/skills.key;	#绑定的证书私钥
}

upstream tomcat {	#upstream为指令域，这是在写一段指令域的配置，里面包含了IP地址和对应端口号
server 10.4.220.103:443;
server 10.4.220.104:443;
}

server {
listen 80;
server_name tomcat.skills.lan;

rewrite ^(.*) https://$server_name$1 permanent;
}

server {
listen 443 ssl http2;
server_name tomcat.skills.lan;
ssl_certificate /etc/ssl/skills.crt;
ssl_certificate_key /etc/ssl/skills.key;
location / {
proxy_pass https://tomcat;
}

}
```
注意：前面三段server是1小题的要求，后面三段2小题的要求
### 5、创建ng.conf中指定的主页的新目录并编辑主页内容
mkdir /var/nginx #创建ng<br />vim /var/nginx/index.html 写入主页内容hellonginx，结果如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1690045075329-c36e9fc4-c1d4-4ff4-89d7-9c30318c2af1.png#averageHue=%2314100e&clientId=uc9350964-e67f-4&from=paste&height=120&id=u195669a5&originHeight=120&originWidth=921&originalType=binary&ratio=1&rotation=0&showTitle=false&size=14584&status=done&style=none&taskId=ub087af00-6206-4db5-ad1e-36d136d3fc2&title=&width=921)

1小题测试<br />systemctl restart nginx.service  #重新启动nginx<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1690045210908-aaf53f5f-9189-4c56-9ebe-4414da3bdb71.png#averageHue=%230e0c0a&clientId=uc9350964-e67f-4&from=paste&height=484&id=u419aad9a&originHeight=484&originWidth=873&originalType=binary&ratio=1&rotation=0&showTitle=false&size=44579&status=done&style=none&taskId=ue28875ae-d8c7-4130-9c9b-8b1207c4c8f&title=&width=873)<br />上图所示：http可以跳转，https能正常访问，IP则访问不了，提示403错误<br />2小题测试<br />首先是负载均衡<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1690045263599-d1091e2d-c5a4-4a25-8332-c1478ab2cdc7.png#averageHue=%230f0d0b&clientId=uc9350964-e67f-4&from=paste&height=218&id=ub2f029d4&originHeight=218&originWidth=841&originalType=binary&ratio=1&rotation=0&showTitle=false&size=22872&status=done&style=none&taskId=u7ac64d1f-afbf-4575-ad00-27ab5282226&title=&width=841)<br />上图所示：每次访问的服务器都在两台之间互相切换，以此来分摊访问，也就是我们负载均衡应有的效果

下面测试80端口和IP的访问<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1690045327873-950bb787-40a0-48f7-aca7-931025598fdd.png#averageHue=%230f0c0b&clientId=uc9350964-e67f-4&from=paste&height=217&id=ufe8e9784&originHeight=217&originWidth=788&originalType=binary&ratio=1&rotation=0&showTitle=false&size=19083&status=done&style=none&taskId=u9af1e6a7-59b8-4f10-a14d-426ecc78937&title=&width=788)<br />上图所示：通过80也就是http访问可以跳转，使用域名无法访问。
