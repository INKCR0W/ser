# 3、ansible服务 √

## **<font style="color:rgb(0,0,0);">题目：</font>**
<font style="color:rgb(0,0,0);">1、在linux1上安装系统自带的ansible-core，作为ansible控制节点。 </font>

<font style="color:rgb(0,0,0);">linux2-linux9 作为 ansible 的受控节点。</font>

## <font style="color:rgb(0,0,0);">配置步骤：</font>
yum install ansible-core*  #安装ansible



vi /etc/ansible/hosts #



[server]

10.4.220.101



[nodes]

10.4.220.102

10.4.220.103

10.4.220.104



ansible nodes -m ping 



> 更新: 2024-05-09 11:21:29  
> 原文: <https://www.yuque.com/gengmouren-1f9qn/whktvz/fxwv3reoewn4bogb>