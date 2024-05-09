---
sidebar_position: 5
---

## **题目：**
 在windows3-windows5 的 C 分区分别划分2GB 的空间，创建NTFS主 分区，驱动器号为D。 配置 windows3 为 DFS 服务器，命名空间为 dfsroot，文件夹为 pictures，存储在D:\dfs，所有用户都具有读写权限；实现windows4 的D:\pics 和windows5 的D:\images 同步。配置windows4 的dfs IPv4 使用34567 端口；限制所有服务的IPv4 动态rpc端口从10000开始，共2000个端口号。  
## 配置步骤：
### 创建NTFS主分区
windows3-winodws5，压缩2GB空间后，依次如下图创建
![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714284341688-bf874cab-0dc0-4ca6-a00a-ae6dfd58751d.png#averageHue=%23f6f5f5&clientId=uae6f84ce-0418-4&from=paste&height=799&id=ucae0d168&originHeight=799&originWidth=1680&originalType=binary&ratio=1&rotation=0&showTitle=false&size=235148&status=done&style=none&taskId=ua6331437-91d3-4e88-bf1a-16b8d06fd23&title=&width=1680)
注意 ！！！1、若创建的分区不是主分区，需删除把空间空闲出来后，打开cmd执行
diskpart
select disk 0
create partition primary
再回到磁盘管理界面，进行格式化并分配盘符

2、若D盘符被其他盘占用，可以把其他盘重新分配盘符，把D空出来给新磁盘
### 安装DFS服务
windows3安装DFS命名空间、复制、文件服务器、其他两台安装复制与文件服务器即可
![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714285144838-1493dfeb-66ba-4065-a28a-a5a160a7044a.png#averageHue=%23f9f8f8&clientId=uae6f84ce-0418-4&from=paste&height=692&id=u437ac24b&originHeight=692&originWidth=975&originalType=binary&ratio=1&rotation=0&showTitle=false&size=57572&status=done&style=none&taskId=ued5a2dba-bf4f-4bd4-bb5b-1c85da3d93b&title=&width=975)
### 命名空间配置
如下图，未提及的选默认项即可
![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714288988226-d07ff49c-d8aa-4eee-ac6c-a5bc213e2fa8.png#averageHue=%23ededed&clientId=uae6f84ce-0418-4&from=paste&height=694&id=ud9bbd336&originHeight=694&originWidth=1512&originalType=binary&ratio=1&rotation=0&showTitle=false&size=126610&status=done&style=none&taskId=u4638100b-b7b2-4af5-a274-437a3b32686&title=&width=1512)
### 复制组配置
成员添加windows3和windows4,，复制的文件夹按题目要求命名
![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714288407267-ab955867-bfac-4a60-9460-8149fa138533.png#averageHue=%23ececeb&clientId=uae6f84ce-0418-4&from=paste&height=366&id=ub0c4974a&originHeight=366&originWidth=1660&originalType=binary&ratio=1&rotation=0&showTitle=false&size=81537&status=done&style=none&taskId=u314c63d8-bf70-4278-ba47-206b8be1e06&title=&width=1660)
记得发布到命名空间，服务器文件夹选pics
![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714288898627-9d8449ee-d33c-4c15-9a09-924d60227a9d.png#averageHue=%23efefef&clientId=uae6f84ce-0418-4&from=paste&height=432&id=udde5aecb&originHeight=432&originWidth=1675&originalType=binary&ratio=1&rotation=0&showTitle=false&size=71781&status=done&style=none&taskId=u7da12e4d-72f1-41fd-b5e2-3b8de1657e8&title=&width=1675)
### 指定端口
指定34567 端口配置及测试
![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1713879418123-1ea6374d-fd70-4e06-b830-d604f8e9b907.png#averageHue=%23110f0e&clientId=u96ccc815-7142-4&from=paste&height=589&id=zMH5o&originHeight=663&originWidth=802&originalType=binary&ratio=1.125&rotation=0&showTitle=false&size=68939&status=done&style=none&taskId=u9f565f11-427a-471f-ac45-e29d8bddbe7&title=&width=712.8888888888889)
### 限制端口
限制端口配置及测试
![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714294889736-883c4d53-e24e-4d61-9deb-e5472ccf4c90.png#averageHue=%231c1c1b&clientId=u5d3dd643-d939-4&from=paste&height=625&id=u1939c927&originHeight=625&originWidth=1170&originalType=binary&ratio=1&rotation=0&showTitle=false&size=46937&status=done&style=none&taskId=uc665f618-c0f9-4d81-867b-c5709c1bd9f&title=&width=1170)
