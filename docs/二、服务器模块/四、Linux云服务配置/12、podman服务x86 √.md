---
sidebar_position: 12
---

## **题目：**
请采用 podman，实现容器虚拟化技术。 
1、在 linux3 上安装 podman，导入 rockylinux-9.tar 镜像。 创建名称为 skills 的容器，映射本机的 8000 端口到容器的 80 端口， 在容器内安装 httpd，默认网页内容为“HelloPodman”。 
2、配置 https 访问的私有仓库，登录用户和密码均为 admin。导入registry.tar 镜像，创建名称为 registry 的容器。 
3、修 改 rockylinux 镜像的tag为linux3.skills.lan:5000/rockylinux:9，上传该镜像到私有仓库。
扩展
podman exec -it skills bash

## 配置步骤：
## 1小题
### 1.1 安装podman
yum install podman* -y
systemctl enable --now podman
### 1.2 导入镜像
podman load -i rockylinux-9.tar
podman images  #查看导入的镜像信息
![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1706090679574-18dfe548-3992-4aee-9f9a-92f29b52ba03.png#averageHue=%23320c26&clientId=u0c7e01e1-0e05-4&from=paste&height=80&id=vkvyA&originHeight=80&originWidth=736&originalType=binary&ratio=1&rotation=0&showTitle=false&size=26386&status=done&style=none&taskId=u7353ca89-cf49-47c0-ac9c-eb1d128d100&title=&width=736)
### 1.3 创建容器
podman run -itd -p 8000:80 --name skills 210996f98b85 /usr/sbin/init
注意！必须加/usr/sbin/init 否则容器内的systemctl用不了
### 1.4 进入容器，进行相应配置
podman exec -it skills bash 

配置http网络源
![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1706090538417-fd36d7c5-07fb-474e-82b1-7d469399fbd7.png#averageHue=%23310b25&clientId=u0c7e01e1-0e05-4&from=paste&height=242&id=u59562248&originHeight=242&originWidth=568&originalType=binary&ratio=1&rotation=0&showTitle=false&size=33901&status=done&style=none&taskId=ub4553832-57ae-4e19-b5c9-2f2e54c3f38&title=&width=568)
yum clean all 
yum makecache
### 1.5 在容器中安装httpd，进行相应配置
yum install httpd
echo "Helledocker" > /var/www/html/index.html
httpd -k start
curl localhost

## 2小题
### 2.1 配置私有仓库
vim /etc/containers/registries.conf
prefix = "linux3.skills.lan:5000"  #指定镜像前缀
location = "linux.skills.lan:5000" #指定获取地址
insecure = true #允许通过http协议获取镜像

systemctl daemon-reload    #重新加载所有 systemd 守护程序配置文件
systemctl restart podman    #重启启动podman，私有仓库的配置生效
### 2.2 导入registry镜像
podman load -i registry.tar


mkdir /podman/admin -p    #登录用户密码存放点
mkdir /podman/registry      #镜像存放点
mkdir /podman/cert	     #证书存放点
htpasswd -Bc /podman/admin/htpasswd admin	#设置htpasswd账户密码
cat /etc/ssl/cacert.pem >> /etc/pki/tls/certs/ca-bundle.crt
![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1706188454651-18a2c4b0-5e91-4667-b5d3-a6535f3e1479.png#averageHue=%23310b25&clientId=u64221790-2036-4&from=paste&height=107&id=u9ec7caab&originHeight=107&originWidth=747&originalType=binary&ratio=1&rotation=0&showTitle=false&size=28754&status=done&style=none&taskId=ub3a5a635-fb09-4ae3-91d5-1237b63d496&title=&width=747)

```
podman run -itd --name registry \
-p 5000:5000 \      #将主机的5000端口映射到容器的5000端口
-v /podman/registry/:/var/lib/registry:z \ #将主机的/podman/registry/挂载到容器的/var/lib/registry，并启用挂载的权限检查。
-v /podman/admin/:/auth:z \		#将主机的/podman/admin/目录挂载到容器的/auth目录，并启用挂载的权限检查。
-e "REGISTRY_AUTH=htpasswd" \  #设置私有仓库的身份验证类型为htpasswd。
-e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \	#设置私有仓库的身份验证域
-e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \   #指定htpasswd文件路径。
-v /podman/cert/:/certs:z \  #将主机的/podman/cert/目录挂载到容器的/certs目录，并启用挂载的权限检查。
-e "REGISTRY_HTTP_TLS_CERTIFICATE=/certs/skills.crt" \ #设置环境变量，指定证书路径
-e "REGISTRY_HTTP_TLS_KEY=/certs/skills.key" \ #设置环境变量，指定私钥的路径
-e REGISTRY_COMPATIBILITY_SCHEMA1_ENABLED=true \ #启用兼容性模式，支持旧版本的镜像
-e REGISTRY_STORAGE_DELETE_ENABLED=true \  #启用删除镜像功能，允许删除镜像
registry
```
podman tag 210996f98b85 linux3.skills.lan:5000/rockylinux:9  #修改标签
podman login -u admin -p admin linux3.skills.lan:5000            #登录到私有仓库
podman push --tls-verify=false linux3.skills.lan:5000/rockylinux:9 #上传  rockylinux:9本地镜像到linux3.skills.lan:5000仓库

curl -k -u "admin:admin" [https://linux3.skills.lan:5000/v2/_catalog](https://linux3.skills.lan:5000/v2/_catalog)

podman exec -it registry /bin/sh
ls certs
ls auth
ls /var/lib/registry/v2/repos/rocky



