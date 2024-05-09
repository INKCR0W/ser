---
sidebar_position: 9
---

## **题目：**
 任务描述：请采用 iSCSI，实现故障转移。<br />1、在 windows7 上安装 iSCSI 目标服务器，并新建 iSCSI 虚拟磁盘，存 储位置为 C:\iscsi；虚拟磁盘名称分别为 Quorum 和 Files，磁盘大小为动态扩展，分别为 1GB 和 5GB，目标名称为 win，访问服务器为 windows8 和 windows9，实行 CHAP 双向认证，Target 认证用户名和 密码分别为 IncomingUser 和 IncomingPass，Initiator 认证用户名 和密码分别为 OutgoingUser 和 OutgoingPass。目标 iqn 名称为 iqn.2008-01.lan.skills:server，使用 IP 地址建立目标。发起程序 iqn 名称分别为 iqn.2008-01.lan.skills:client1 和 iqn.2008- 01.lan.skills:client2。 <br />2、在 windows8 和 windows9 上安装多路径 I/O ，10.4.210.0 和 10.4.211.0 网络为 MPIO 网络，连接 windows7 的虚拟磁盘 Quorum 和 Files，初始化为 GPT 分区表，创建 NTFS 主分区，驱动器号分别为 M 和 N。 <br />3、配置 windows8 和 windows9 为故障转移群集；10.4.212.0 网络为心跳网络。<br />4、在 windows8 上创建名称为cluster 的群集，其IP 地址为 10.4.210.70。 在 windows9 上配置文件服务器角色，名称为 clusterfiles，其 IP 地 址为 10.4.210.80。为 clusterFiles 添加共享文件夹，共享协议采用“SMB”，共享名称为 clustershare，存储位置为 N:\share，NTFS 权 限为仅域管理员和本地管理员组具有完全控制权限，域其他用户具有 修改权限；共享权限为仅域管理员具有完全控制权限，域其他用户具 有更改权限。 

配置 windows8 和 windows9 为故障转移群集；其中有两张网卡，网络分别是10.4.210.0和10.4.212.0，请把10.4.212.0网络配置为心跳网络。<br />MPIO网络：<br />MPIO（Multipath I/O）是一种用于实现存储设备多路径冗余和负载均衡的技术。<br />MPIO有以下几个方面的功能：<br />1. 多路径冗余：MPIO允许服务器通过多个独立的物理路径连接到存储设备。如果其中一个路径出现故障，MPIO可以自动切换到其他可用的路径，确保数据的连续性和可用性，从而提高系统的冗余性。<br />2. 负载均衡：MPIO可以将I/O请求分布到多个路径上，实现负载均衡。通过同时使用多个路径进行数据传输，可以提高存储系统的吞吐量和响应速度。MPIO可以根据路径的可用带宽、延迟和负载情况等因素，动态地选择最佳的路径来处理I/O请求，从而实现负载均衡。<br />3. 故障检测和恢复：MPIO可以检测路径的故障，并自动切换到其他可用的路径。当一个路径出现故障时，MPIO可以快速检测到，并将I/O请求重新定向到其他正常的路径上，从而确保数据的连续性和可用性。
## 配置步骤：
