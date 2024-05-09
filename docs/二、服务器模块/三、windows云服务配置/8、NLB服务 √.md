---
sidebar_position: 8
---

## **题目：**
配置windows5 和windows6 为NLB服务器。 <br />1、windows5 群集优先级为5，windows6 群集优先级为6，群集IPv4地 址为10.4.210.60/24，群集名称为www.skills.lan，采用多播方式。 <br />2、配置windows5 为 web 服务器，站点名称为 www，网站的最大连接数 为10000，网站连接超时为60s，网站的带宽为100Mbps。<br />3、共享网页文件、共享网站配置文件和网站日志文件分别存储到 windows1 的 D:\FilesWeb\Contents 、 D:\FilesWeb\Configs 和 D:\FilesWeb\Logs。网站主页index.html 内容为"HelloNLB"。<br />4、使用W3C记录日志，每天创建一个新的日志文件，日志只允许记录日期、时间、客户端IP地址、用户名、服务器IP地址、服务器端口号。 网站仅绑定https，IP 地址为群集地址，仅允许使用域名加密访问， 使用“计算机副本”证书。<br />5、配置windows6 为 web 服务器，要求采用共享windows5配置的方式， 使用“计算机副本”证书。
## 配置步骤：
### 1小题
1、确保网卡已添加，并设置了相应IP<br />2、windows5、windows6安装web服务（添加IP和域名限制组件）、网络负载均衡功能<br />3、window5打开网络负载平衡管理器，进行下图配置，windows6刷新即可，无需配置：<br />注：windows6的接口是二次添加<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714308622075-6df05135-6ba7-4bba-967a-b4028413fda0.png#averageHue=%23fbfafa&clientId=u6b1fda86-88f5-4&from=paste&height=574&id=u6d246747&originHeight=574&originWidth=1428&originalType=binary&ratio=1&rotation=0&showTitle=false&size=103666&status=done&style=none&taskId=u4946a60c-9f3b-4d77-85eb-144b27ebac5&title=&width=1428)
### 3小题
#### 文件夹创建
按要求在windows1创建文件夹--设置共享--新建主页文件--写入内容-改为html格式<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714309355964-61d6322a-1d69-47c2-9eda-dc8d0420d5d0.png#averageHue=%23fafafa&clientId=u1cb23470-3b1d-4&from=paste&height=673&id=u7b6317aa&originHeight=673&originWidth=1718&originalType=binary&ratio=1&rotation=0&showTitle=false&size=58239&status=done&style=none&taskId=uf210fe3d-1856-4516-ae8d-efa2d72722c&title=&width=1718)
### 2小题
#### 新建网站
![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714309566499-8f13832a-d533-458c-8a9f-f6689a9792ae.png#averageHue=%23f2f2f1&clientId=u1cb23470-3b1d-4&from=paste&height=638&id=ub6178b70&originHeight=638&originWidth=1234&originalType=binary&ratio=1&rotation=0&showTitle=false&size=130140&status=done&style=none&taskId=u46444a96-8d10-4e24-8118-9d0142e93d9&title=&width=1234)
#### 网站限制
![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714310046708-bafb73b2-d0f0-4385-a378-b7f61215a197.png#averageHue=%23f6f5f5&clientId=u1cb23470-3b1d-4&from=paste&height=721&id=u7f4a9207&originHeight=721&originWidth=1375&originalType=binary&ratio=1&rotation=0&showTitle=false&size=188693&status=done&style=none&taskId=u91b53080-bdd9-45ff-b857-e20b9f989cc&title=&width=1375)
### 4小题
#### 日志记录
![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714310447434-831db8dd-f8ea-47e6-9236-a359f5283012.png#averageHue=%23fafaf9&clientId=u1cb23470-3b1d-4&from=paste&height=804&id=pXxXc&originHeight=804&originWidth=1631&originalType=binary&ratio=1&rotation=0&showTitle=false&size=169509&status=done&style=none&taskId=u5dcb58b0-d3a3-45e7-b900-a1bc22df6e8&title=&width=1631)
#### https配置
导入证书、添加443端口绑定、删除80端口的绑定<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714310693800-39a32398-d872-46df-a9c4-c04cac125e6b.png#averageHue=%23f3f3f3&clientId=u1cb23470-3b1d-4&from=paste&height=878&id=u2a7ee618&originHeight=878&originWidth=1593&originalType=binary&ratio=1&rotation=0&showTitle=false&size=180978&status=done&style=none&taskId=ufde45e4b-3fb4-4986-80b6-5584ab79816&title=&width=1593)
#### dns解析
windows1 做www与210.60的正反向记录<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714311017436-48fdb446-f426-40b7-9452-56c67055a442.png#averageHue=%23f4f3f2&clientId=u1cb23470-3b1d-4&from=paste&height=655&id=u98d7eb8e&originHeight=655&originWidth=1036&originalType=binary&ratio=1&rotation=0&showTitle=false&size=201255&status=done&style=none&taskId=u580a63dc-8978-4622-bf51-c7a9633ce4a&title=&width=1036)
#### 访问主页
![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714311060787-9091bd9d-a92c-4ac1-8daa-4b5a8d7fb35b.png#averageHue=%23fbfafa&clientId=u1cb23470-3b1d-4&from=paste&height=644&id=u01dc0d19&originHeight=644&originWidth=1292&originalType=binary&ratio=1&rotation=0&showTitle=false&size=17568&status=done&style=none&taskId=ufe5e9af2-41df-4aff-838f-196849d8ac7&title=&width=1292)
#### 仅域名访问
![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714311214225-0fcce670-6236-437f-8aa1-7971abf280d8.png#averageHue=%23c9b392&clientId=u1cb23470-3b1d-4&from=paste&height=469&id=ufb91a10d&originHeight=469&originWidth=1635&originalType=binary&ratio=1&rotation=0&showTitle=false&size=148721&status=done&style=none&taskId=uadcdf85e-82bf-437d-8692-5815d10195b&title=&width=1635)
### 5小题
#### 共享配置
加密密码需包含字符大小写<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714444843310-84729576-a80c-404b-ace7-d8fd3e7dcdad.png#averageHue=%23f7f6f6&clientId=u4cacad7c-7195-4&from=paste&height=833&id=u2495870f&originHeight=878&originWidth=1918&originalType=binary&ratio=1&rotation=0&showTitle=false&size=253936&status=done&style=none&taskId=u41c306ff-e169-4db3-b24c-a6daa57a639&title=&width=1820)<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714445304966-b1bcb9e5-b314-4b4e-878d-186c07032d20.png#averageHue=%23f9f9f8&clientId=u4cacad7c-7195-4&from=paste&height=611&id=u746db609&originHeight=611&originWidth=1899&originalType=binary&ratio=1&rotation=0&showTitle=false&size=187524&status=done&style=none&taskId=uc5196267-8ed6-4e70-998e-34d438a5cd0&title=&width=1899)<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/33622884/1714445751874-dba70e7e-ec15-4d7c-85db-78f6913ab639.png#averageHue=%23f4f4f3&clientId=u4cacad7c-7195-4&from=paste&height=692&id=u19442de4&originHeight=692&originWidth=1918&originalType=binary&ratio=1&rotation=0&showTitle=false&size=211582&status=done&style=none&taskId=u63bc3013-8919-4eda-980c-8f6419f11f9&title=&width=1918)<br />上图可先导win6证书再配置共享，这样顺序就合理得多
