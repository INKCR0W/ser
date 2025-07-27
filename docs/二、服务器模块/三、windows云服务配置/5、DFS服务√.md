# 5、DFS服务 √

## **<font style="color:rgb(0,0,0);">题目：</font>**
 在windows3-windows5 的 C 分区分别划分2GB 的空间，创建NTFS主 分区，驱动器号为D。 配置 windows3 为 DFS 服务器，命名空间为 dfsroot，文件夹为 pictures，存储在D:\dfs，所有用户都具有读写权限；实现windows4 的D:\pics 和windows5 的D:\images 同步。配置windows4 的dfs IPv4 使用34567 端口；限制所有服务的IPv4 动态rpc端口从10000开始，共2000个端口号。  

## <font style="color:rgb(0,0,0);">配置步骤：</font>
### 创建NTFS主分区
windows3-winodws5，压缩2GB空间后，依次如下图创建

![1714284341688-bf874cab-0dc0-4ca6-a00a-ae6dfd58751d.png](./img/xT-Bl_8YaGX5kwRO/1714284341688-bf874cab-0dc0-4ca6-a00a-ae6dfd58751d-497520.png)

<details class="lake-collapse"><summary id="u9c461690"><span class="ne-text">注意 ！！！</span></summary><p id="u1b30a954" class="ne-p"><span class="ne-text">1、若创建的分区不是主分区，需删除把空间空闲出来后，打开cmd执行</span></p><p id="uda8786fe" class="ne-p"><span class="ne-text" style="font-size: 16px">diskpart</span></p><p id="u098bb1ef" class="ne-p"><span class="ne-text" style="font-size: 16px">select disk 0</span></p><p id="ucc4672ab" class="ne-p"><span class="ne-text" style="font-size: 16px">create partition primary</span></p><p id="uc5bd7198" class="ne-p"><span class="ne-text" style="font-size: 16px">再回到磁盘管理界面，进行格式化并分配盘符</span></p><p id="ua0b3770a" class="ne-p"><span class="ne-text" style="font-size: 16px"></span></p><p id="u8b1d9019" class="ne-p"><span class="ne-text" style="font-size: 16px">2、若D盘符被其他盘占用，可以把其他盘重新分配盘符，把D空出来给新磁盘</span></p></details>
### 安装DFS服务
windows3安装DFS命名空间、复制、文件服务器、其他两台安装复制与文件服务器即可

![1714285144838-1493dfeb-66ba-4065-a28a-a5a160a7044a.png](./img/xT-Bl_8YaGX5kwRO/1714285144838-1493dfeb-66ba-4065-a28a-a5a160a7044a-067575.png)

### 命名空间配置
如下图，未提及的选默认项即可

![1714288988226-d07ff49c-d8aa-4eee-ac6c-a5bc213e2fa8.png](./img/xT-Bl_8YaGX5kwRO/1714288988226-d07ff49c-d8aa-4eee-ac6c-a5bc213e2fa8-856368.png)

### 复制组配置
成员添加windows3和windows4,，复制的文件夹按题目要求命名

![1714288407267-ab955867-bfac-4a60-9460-8149fa138533.png](./img/xT-Bl_8YaGX5kwRO/1714288407267-ab955867-bfac-4a60-9460-8149fa138533-264588.png)

记得发布到命名空间，服务器文件夹选pics

![1714288898627-9d8449ee-d33c-4c15-9a09-924d60227a9d.png](./img/xT-Bl_8YaGX5kwRO/1714288898627-9d8449ee-d33c-4c15-9a09-924d60227a9d-357031.png)

### 指定端口
指定34567 端口配置及测试

![1713879418123-1ea6374d-fd70-4e06-b830-d604f8e9b907.png](./img/xT-Bl_8YaGX5kwRO/1713879418123-1ea6374d-fd70-4e06-b830-d604f8e9b907-531604.png)

### 限制端口
限制端口配置及测试

![1714294889736-883c4d53-e24e-4d61-9deb-e5472ccf4c90.png](./img/xT-Bl_8YaGX5kwRO/1714294889736-883c4d53-e24e-4d61-9deb-e5472ccf4c90-921521.png)



> 更新: 2024-04-29 08:45:44  
> 原文: <https://www.yuque.com/gengmouren-1f9qn/whktvz/efofb97qnnwrmym1>