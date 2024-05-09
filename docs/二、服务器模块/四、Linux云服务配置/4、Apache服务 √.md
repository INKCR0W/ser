---
sidebar_position: 4
---

## ** 题目：**
1、配置 linux1 为 Apache2 服务器，使用 skills.lan 或 any.skills.lan （any 代表任意网址前缀，用 linux1.skills.lan 和 web.skills.lan 测试）访问时，自动跳转www.skills.lan。禁止使用 IP 地址访问， 默认首页文档/var/www/html/index.html 的内容为"HelloApache"。 <br />2、把/etc/pki/tls/skills.crt 证书文件和/etc/pki/tls/skills.key 私钥文件转换成含有证书和私钥的/etc/pki/tls/skills.pfx 文件； 然后把 /etc/pki/tls/skills.pfx 转 换 为 含 有 证 书 和 私 钥 的 /etc/pki/tls/skills.pem 文件，再从/etc/pki/tls/skills.pem 文 件 中 提 取 证 书 和 私 钥 分 别 到 /etc/pki/tls/apache.crt 和 /etc/pki/tls/apache.key。<br />3、客户端访问 Apache 服务时，必需有 ssl 证书。
## 配置步骤：
## 1小题
### 1、安装httpd服务
yum install httpd mod_ssl -y    #mod_ssl为2-3小题所用
### 2、修改ssl.conf配置文件
```
#http将所有访问 linux1.skills.lan 或 any.skills.lan 的请求重定向到 https://www.skills.lan
<VirtualHost *:80>
ServerName linux1.skills.lan
ServerAlias any.skills.lan
rewriteengine on
rewriterule ^/(.*) https://www.skills.lan
</VirtualHost>
#https启用证书并绑定
<VirtualHost *:443>
ServerName linux1.skills.lan
SSLEngine on
SSLCertificateFile /etc/ssl/skills.crt
SSLcertificatekeyFile /etc/ssl/skills.key
</VirtualHost>
#http重定向到403错误页面
<VirtualHost *:80>
servername 10.4.220.101
redirect 403 /
</virtualHost>
#https重定向到403错误页面
<VirtualHost *:443>
servername 10.4.220.101
redirect 403
</virtualHost>
```
### 3、写入主页文件内容
echo HelloApache >> /var/www/html/index.html 写入内容
###  4、修改dns解析文件
正向区域文件中加入*与10.4.220.101的对应记录，反向不用加<br />*       A       10.4.220.101<br />systemctl restart named<br />注意：在hosts文件中写对应关系只能在本机正常测试
## 2小题
key和crt 转为pfx<br />openssl pkcs12 -export -out skills.pfx -inkey skills.key -in skills.crt 

pfx转为pem<br /> openssl pkcs12 -in skills.pfx -nodes -out skills.pem

提取pem中的证书为crt和key   <br />openssl x509 -in skills.pem -out apache.crt  <br />openssl rsa -in skills.pem -out apache.key
## 3小题
```
SSLCertificateFile /etc/pki/tls/apache.crt		 #绑定ssl证书
SSLCertificateKeyFile /etc/pki/tls/apache.key  #绑定ssl证书私钥
SSLCACertificateFile /etc/pki/tls/cacert.pem   #绑定ca证书
SSLVerifyClient require    #客户端必须提供有效的证书才能与服务器进行SSL连接
SSLVerifyDepth  10				 #设置为10表示在验证证书链时，最多可以遍历到10层，超过这个深度将拒绝连接。
```
systemctl restart httpd
## 测试
cd /etc/pki/tls  #切换至证书所在目录<br />跳转测试<br /> curl [http://linux1.skills.lan](http://linux1.skills.lan)<br />curl [http://web.skills.lan](http://100.skills.lan)<br />curl [http://100.skills.lan](http://100.skills.lan)<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714053274741-2e4cda88-9bba-4626-9309-29761410fa28.png#averageHue=%23090706&clientId=u52aa31ee-ffc9-4&from=paste&height=652&id=ud104beaa&originHeight=734&originWidth=1183&originalType=binary&ratio=1.125&rotation=0&showTitle=false&size=79289&status=done&style=none&taskId=u673be210-d25f-4ffc-b984-6019f87bb88&title=&width=1051.5555555555557)<br />必需证书测试<br />curl [https://www.skills.lan](https://www.skills.lan)<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714053398579-671d22ac-7492-4942-857c-4728f7c33714.png#averageHue=%23120f0d&clientId=u52aa31ee-ffc9-4&from=paste&height=62&id=uc315795f&originHeight=70&originWidth=1187&originalType=binary&ratio=1.125&rotation=0&showTitle=false&size=14434&status=done&style=none&taskId=uc9a25b17-8ff0-4b59-b493-f914ae27b05&title=&width=1055.111111111111)<br />必需证书测试<br />curl [https://www.skills.lan](https://www.skills.lan) --cert skills.pem <br />curl [https://www.skills.lan](https://www.skills.lan) --cert skills.crt --key skills.key<br />curl [https://www.skills.lan](https://www.skills.lan) --cert apache.crt --key apache.key<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714053501155-d19272df-6dc6-4c55-b576-561e17d9e44e.png#averageHue=%230e0c0a&clientId=u52aa31ee-ffc9-4&from=paste&height=164&id=u4c5a1f8a&originHeight=184&originWidth=1187&originalType=binary&ratio=1.125&rotation=0&showTitle=false&size=30295&status=done&style=none&taskId=ue75503ca-f82a-4b94-8d9d-662130918c9&title=&width=1055.111111111111)
