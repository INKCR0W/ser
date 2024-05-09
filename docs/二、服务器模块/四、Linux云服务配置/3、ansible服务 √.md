---
sidebar_position: 3
---

## **题目：**
1、在linux1上安装系统自带的ansible-core，作为ansible控制节点。 
linux2-linux9 作为 ansible 的受控节点。
## 配置步骤：
yum install ansible-core.x86_64  #安装ansible

vi /etc/ansible/hosts #

[server]
10.4.220.101

[nodes]
10.4.220.102
10.4.220.103
10.4.220.104

ansible nodes -m ping 
