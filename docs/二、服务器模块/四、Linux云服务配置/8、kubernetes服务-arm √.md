---
sidebar_position: 8
---

## **题目：**
请采用 kubernetes 和 containerd，管理容器。 <br />1、在 linux5-linux7 上安装 containerd 和 kubernetes，linux5 作为master node ， linux6 和 linux7 作 为 work node ； 使 用 containerd.sock 作 为 容 器 runtime-endpoint 。 pod 网 络 为10.244.0.0/16，services 网络为 10.96.0.0/16。 <br />**2、**master 节点配置 calico 作为网络组件。 <br />3、导入 nginx.tar 镜像，主页内容为“HelloKubernetes”。用该镜像创 <br />建一个名称为 web 的 deployment（部署），副本数为 2；为该 deployment 创 <br />建一个类型为 nodeport 的 service，port 为 80，targetPort 为 80， <br />nodePort 为 30000。

## 配置步骤：
## 准备工作--所有节点 
1、禁用交换分区，保证kubelet正常工作    vi /etc/fstab #修改fstab文件，注释掉交换分区 <br />echo "vm.swappiness = 0" >> /etc/sysctl.conf #使其永久不分配<br />swapoff -a #临时关闭
2、关闭防火墙或放行服务    systemctl stop firewalld #关闭防火墙<br />systemctl disable firewalld #开机不自启<br />除了关闭外，也可以放行相应服务<br />firewall-cmd --permanent --add-service=kadmin
3、关闭 selinux     vi /etc/selinux/config #将enabled改为disabled<br />setenforce 0 #临时关闭
4、修改 hosts 文件（可不做）vi /etc/hosts #节点IP与域名做对应关系<br />linux5	10.4.220.105	linux5.skills.lan<br />linux6	10.4.220.106	linux6.skills.lan<br />linux7	10.4.220.107	linux7.skills.lan
5、确保 chrony 时间服务同步正常    vi /etc/chonryd.config #编辑chrony服务配置文件<br />systemctl restart chronyd #重启时间服务<br />date #查看当前时间<br />6、证书和证书私钥发送至Linux5
# 1小题
## 第一步：安装kubernetes--所有节点
1、解压并安装所有rpm包    <br />tar -xf  kubernetes.tar<br />cd /kubernetes/packages<br />rpm -Uvh --force --nodeps *.rpm或者yum install *.rpm<br />使用yum可以把其他依赖包一并安装，前提是yum本地源要正常<br />source <(kubectl completion bash)  #配置kubernetes tab补全
## 第二步：调整内核参数--所有节点
写入内容1:    为Kubernetes提供所需的底层支持cat <<EOF | tee /etc/modules-load.d/k.conf<br />overlay<br />br_netfilter<br />EOF<br />modprobe overlay #加载模块<br />modprobe br_netfilter #加载模块
写入内容2：使网络可以正常处理和转发cat <<EOF | tee /etc/sysctl.d/k.conf<br />net.bridge.bridge-nf-call-iptables = 1<br />net.bridge.bridge-nf-call-ip6tables = 1<br />net.ipv4.ip_forward = 1<br />EOF<br />sysctl -p --system #重载内核参数
## 第三步：Kubelet驱动配置--所有节点
vi /etc/sysconfig/kubeletKUBELET_CGROUP_ARGS="--cgroup-driver=systemd"  #Kubelet使用systemd作为cgroup驱动
注配置Kubelet使用systemd作为cgroup驱动，可以确保在不同环境下的兼容性，例如x86和arm<br />在Kubernetes中，cgroup被用来管理Pod和容器的资源使用。<br />systemctl enable --now containerd.service kubelet.service #设为开机自启，并现在启动
## 第四步：导入镜像--所有节点
cd 切换到镜像所在目录<br />ctr -n k8s.io image import +  镜像名  --platform=linux/arm64   #单个导入镜像到指定的命名空间<br />for i in *;do ctr -n k8s.io i import $i --platform=linux/arm64 ;done #批量导入<br />ctr -n k8s.io i list -q  #列出命名空间为 "k8s.io" 的所有镜像ID，镜像位置与config.toml对应
注kubeadm config images list #查看所需镜像列表    <br />ctr namespace ls 查看所有命名空间
## 第五步：配置containerd--所有节点
修改containerd配置文件    <br />containerd config default | tee > /etc/containerd/config.toml #生成containerd配置文件
```
修改1：[plugins."io.containerd.grpc.v1.cri"]
sandbox_image = "reqistry.aliyuncs.com/google_containers/pause:3.9"与要安装的pause版本对应

修改2：[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
SystemdCgroup = true
```

配置containerd.sock 作为容器 runtime-endpoint<br />crictl config runtime-endpoint /run/containerd/containerd.sock

systemctl restart containerd.service kubelet.service #重启contnainerd和kubelet
## 第六步：初始化集群--主节点
kubeadm config print init-defaults > kubA.yaml导出初始化配置信息
```
advertiseAddress: 10.4.220.105 #主节点IP
imagePullPolicy: Never #本地拉取镜像
name: linux5 #主节点主机名/域名/IP
kubernetesVersion: 1.27.1 #指定kubernetes具体版本
serviceSubnet: 10.96.0.0/16
podSubnet: 10.244.0.0/16
```
注imagePullPolicy可以有三种选项

1. **Always**：此策略表示每次Pod启动时，无论节点上是否已经存在指定版本的镜像，都会尝试从远程镜像仓库拉取最新的镜像。如果本地已经存在该镜像，也会重新拉取并覆盖本地的镜像。
2. **IfNotPresent**：这是imagePullPolicy的默认值。在此策略下，如果节点上不存在指定的镜像，则将从远程仓库拉取。如果节点上已经存在该镜像，则不会拉取新的镜像。
3. **Never**：此策略意味着Pod将永远不会从远程镜像仓库拉取镜像，只会使用节点上已经存在的镜像。如果节点上不存在指定的镜像，则容器无法启动。

kubeadm init --config=kubA.yaml #选择该文件进行初始化集群
初始化完成后手动复制执行以下内容mkdir -p $HOME/.kube<br />sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config<br />sudo chown $(id -u):$(id -g) $HOME/.kube/config
## 第六步：加入集群--工作节点
1、主节点初始化完成后会生成类似下方的指令，把该内容复制到工作节点执行，即可加入集群<br />kubeadm join 10.4.220.105:6443 --token abcdef.0123456789abcdef \<br />> --discovery-token-ca-cert-hash sha256:8d987951412a23a94c7a131c99d39d0b0dd2596d7c80850033da30ad3874f367

kubectl get nodes #在主节点执行，可以查看加入情况<br />kubectl get pods --all-namespaces #查看所有镜像运行情况
# 2小题
## 第一步：创建calico网络--主节点
cat kubA.yaml<br />找到podSubnet: 将网络信息复制

cd /kubernetes/calico/<br />vi calico.yaml #在该文件下分别搜索 k8s,bgp和no effect<br />在k8s,bgp下添加两行，ensp1s0为网卡名称<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1704784385501-8c71353c-eaeb-4b02-83fc-0b225c843a63.png#averageHue=%23310b25&clientId=u14b596dc-9e8a-4&from=paste&height=203&id=gD61W&originHeight=203&originWidth=803&originalType=binary&ratio=1&rotation=0&showTitle=false&size=49377&status=done&style=none&taskId=u30d489b5-461a-4562-be08-19ccdd3d21e&title=&width=803)<br />在no effect下取消注释并进行以下修改，10.244.0.0为，kubA中的pods网络信息<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1704784529827-64224441-fa7a-4e9f-9c8b-e922001a7b59.png#averageHue=%23310b25&clientId=u14b596dc-9e8a-4&from=paste&height=109&id=u8966d942&originHeight=109&originWidth=866&originalType=binary&ratio=1&rotation=0&showTitle=false&size=31631&status=done&style=none&taskId=u6cb06c27-295d-49aa-b26d-02fdd9b9e84&title=&width=866)
注意！！！！23年国赛涉及的calico相关的镜像版本如下<br />cni-v3.25.0.tar<br />kube-controllers-v3.25.0.tar<br />node-v3.25.0.tar

若calico.yaml文件中的版本与以上不同，需要自己修改为3.25.0，arm架构的修改为3.25.0-arm<br />kubectl apply -f calico.yaml 创建网络组件脚本<br /> <br />for i in *;do ctr -n k8s.io i import $i --platform=linux/arm64 ;done #导入网络组件所需的镜像文件 **#所有节点**

过一会儿后<br />kubectl get nodes #查看节点信息，可以看到所有节点就绪<br />kubectl get pods --all-namespaces #可以看到所有镜像已运行
# 3小题兼1小题部分要求
## 第一步：导入nginx镜像--所有节点
ctr -n k8s.io image import nginx.tar
## 第二步：创建部署--主节点
1、kubectl create deployment web --image=nginx --replicas=2   #名称为web 映像为nginx 副本数为2<br />![截图 2023-07-11 21-54-28.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1689083701241-075abcd1-be2f-4a42-a3f0-19d4104a972a.png#averageHue=%230c0a08&clientId=u4a084733-cd49-4&from=drop&id=nN3Ox&originHeight=87&originWidth=1199&originalType=binary&ratio=1&rotation=0&showTitle=false&size=20138&status=done&style=none&taskId=u9608ec35-fe60-4465-bc5e-0b1c29a14b0&title=)2、kubectl edit deployment web #使用系统编辑器打开名为web的部署，<br />![截图 2023-07-11 21-57-15.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1689083849741-91a4c8f8-9bc2-4a5e-9574-a1ffd4973b8d.png#averageHue=%230a0907&clientId=u4a084733-cd49-4&from=drop&id=u11ef7b6a&originHeight=79&originWidth=1043&originalType=binary&ratio=1&rotation=0&showTitle=false&size=16638&status=done&style=none&taskId=u46de7775-7243-4a9d-81b4-351ac0818b4&title=)修改imagepullpolicy处，IfNotPresent的作用是指定优先使用本机缓存的镜像，如果本地没有在从仓库拉取<br />![截图 2023-07-11 21-56-54.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1689083855988-e06a206f-4ca5-470a-9053-b509234fc819.png#averageHue=%23080706&clientId=u4a084733-cd49-4&from=drop&id=u47543472&originHeight=360&originWidth=1043&originalType=binary&ratio=1&rotation=0&showTitle=false&size=42263&status=done&style=none&taskId=u31991aca-f977-4a0a-97e0-cc1ff0a1fa7&title=)3、kubectl get pod #查看pod信息<br />![截图 2023-07-11 21-59-10.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1689084103474-00e4cd82-e82b-477a-94fc-956631ed8f4f.png#averageHue=%230c0a08&clientId=u4a084733-cd49-4&from=drop&id=u2e22d06c&originHeight=157&originWidth=1043&originalType=binary&ratio=1&rotation=0&showTitle=false&size=32799&status=done&style=none&taskId=ubf1a201c-767f-40dd-b7a4-e2171e9704d&title=)
## 第三步：映射端口--主节点
vim /etc/kubernetes/manifests/kube-apiserver.yaml<br />添加：- --service-node-port-range=1-65535<br />![截图 2023-07-11 22-01-30.png](https://cdn.nlark.com/yuque/0/2023/png/33622884/1689084131588-873743a2-e00a-45de-9550-0747ab2677d8.png#averageHue=%230e0c0b&clientId=u4a084733-cd49-4&from=drop&id=ub4667159&originHeight=590&originWidth=1217&originalType=binary&ratio=1&rotation=0&showTitle=false&size=105109&status=done&style=none&taskId=u49549a2d-568d-4252-80b7-ed7eaafd8c1&title=)systemctl restart kubelet.service #重启kubelet.service

kubectl expose deployment web --port=80 --target-port=80 --type=NodePort 

kubectl edit svc web #编辑副本web的配置文件<br />#修改以下字段<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1704787505314-ba64bb47-8952-40c1-917d-29209bda0c10.png#averageHue=%23300a25&clientId=u14b596dc-9e8a-4&from=paste&height=447&id=ub8a7f1c7&originHeight=447&originWidth=663&originalType=binary&ratio=1&rotation=0&showTitle=false&size=73508&status=done&style=none&taskId=u669a4edb-fa6f-4ab8-b636-66181b4baea&title=&width=663)<br />kubectl get svc #查看服务<br />kubectl get pod -o wide #查看服务器pod详情，在哪个节点运行<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1704787597421-b3b47a40-937d-484e-a78c-34560cf0ba8c.png#averageHue=%23310b25&clientId=u14b596dc-9e8a-4&from=paste&height=195&id=u70b43bfb&originHeight=195&originWidth=1152&originalType=binary&ratio=1&rotation=0&showTitle=false&size=61163&status=done&style=none&taskId=u9000697f-f78b-4c74-bbcf-bedb9cf9383&title=&width=1152)
## 第四步：使用kubectl配置nginx--主节点
kubectl exec -it web-79f9c88bdc-djtfh bash #进入Linux6节点<br />echo "HelloKubernetes" > /usr/share/nginx/html/index.html<br />nginx -s reload #重启nginx<br />curl localhost #测试访问情况<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1704789048358-ab7d53eb-8ab0-4934-b5d6-34c509f55e19.png#averageHue=%23310c26&clientId=u14b596dc-9e8a-4&from=paste&height=53&id=ud5a303f1&originHeight=53&originWidth=564&originalType=binary&ratio=1&rotation=0&showTitle=false&size=14420&status=done&style=none&taskId=ua9676b95-fb7e-499f-a105-12ae63b7d0d&title=&width=564)

kubectl exec -it web-79f9c88bdc-vhz4b bash #进入Linux7节点<br />echo "HelloKubernetes" > /usr/share/nginx/html/index.html<br />nginx -s reload #重启nginx<br />curl localhost #测试访问情况<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1704788941342-0cbc5f9d-9738-432d-9af2-04190992d5b1.png#averageHue=%23320c26&clientId=u14b596dc-9e8a-4&from=paste&height=52&id=u7daac201&originHeight=52&originWidth=550&originalType=binary&ratio=1&rotation=0&showTitle=false&size=14402&status=done&style=none&taskId=u726e7065-91ed-430c-958d-83b881520a9&title=&width=550)
## 第五步：测试--主节点
两个节点访问都没问题后，使用主节点测试访问情况<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1704789171131-c998f7df-45e2-424f-bece-1ad5a20931fe.png#averageHue=%23310c26&clientId=u14b596dc-9e8a-4&from=paste&height=239&id=ued879bc9&originHeight=239&originWidth=602&originalType=binary&ratio=1&rotation=0&showTitle=false&size=56919&status=done&style=none&taskId=ub56a9b91-84df-4015-b728-f2b5f800628&title=&width=602)
