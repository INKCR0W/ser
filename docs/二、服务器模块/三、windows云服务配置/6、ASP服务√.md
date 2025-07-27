# 6、ASP服务 √

## **<font style="color:rgb(0,0,0);">题目：</font>**
<font style="color:rgb(0,0,0);">任务描述：请采用 IIS 搭建 web 服务，创建安全动态网站，。 </font>

<font style="color:rgb(0,0,0);">把 windows3 配置为 ASP 网站，网站仅支持 dotnet clr v4.0，站点名称为 asp。 </font>

<font style="color:rgb(0,0,0);">http 和 https 绑定本机与外部通信的 IP 地址，仅允许使用域名访问 （使用“计算机副本”证书模板）。客户端访问时，必需有 ssl 证书 （浏览器证书模板为“管理员”）。 </font>

<font style="color:rgb(0,0,0);">网 站 目 录 为 C:\iis\contents ， 默 认 文 档 index.aspx 内 容 为 "HelloAspx"。 </font>

<font style="color:rgb(0,0,0);">使用 windows5 测试。</font>

## <font style="color:rgb(0,0,0);">配置步骤：</font>
### web服务安装
<font style="color:#DF2A3F;">注意勾选角色服务里的asp和asp.net 4.8</font>

![1714272895324-235e16fd-48ec-4d0f-b0a9-f15542b60d54.png](./img/_-ZEKwHeFJ9mpBof/1714272895324-235e16fd-48ec-4d0f-b0a9-f15542b60d54-824907.png)

### 网站配置
右击新建网站，创建对应目录

![1714273294337-eb5c7227-6bf8-4f57-a0fc-3617b29e14fd.png](./img/_-ZEKwHeFJ9mpBof/1714273294337-eb5c7227-6bf8-4f57-a0fc-3617b29e14fd-149837.png)

导入windows1提供的skills证书

![1714274165657-7a2051b1-1b36-4e8b-8f1f-c98aebf06b1c.png](./img/_-ZEKwHeFJ9mpBof/1714274165657-7a2051b1-1b36-4e8b-8f1f-c98aebf06b1c-822363.png)右击asp网站，选择编辑绑定添加https绑定

![1714273994164-706a6ce2-e32c-4f13-84fc-421f24ba7e40.png](./img/_-ZEKwHeFJ9mpBof/1714273994164-706a6ce2-e32c-4f13-84fc-421f24ba7e40-224275.png)

更改asp网站的应用池为经典<font style="color:rgb(0,0,0);">dotnet clr v4.0</font>

![1698241425514-7ede6f86-dcb1-45f1-8f78-2010bb140079.png](./img/_-ZEKwHeFJ9mpBof/1698241425514-7ede6f86-dcb1-45f1-8f78-2010bb140079-085200.png)

全局处启用ASP父路径

![1698241354067-f9f66142-67d4-45a8-8ffd-f2e298359d96.png](./img/_-ZEKwHeFJ9mpBof/1698241354067-f9f66142-67d4-45a8-8ffd-f2e298359d96-713172.png)

添加MIME类型，使其支持aspx格式的文件

![1698242066273-4482f19d-2626-4c10-b58a-97fafbe100d1.png](./img/_-ZEKwHeFJ9mpBof/1698242066273-4482f19d-2626-4c10-b58a-97fafbe100d1-061225.png)

手动在网站目录下新建index.aspx并添加内容HelloAspx，然后如下图添加默认文档

![1698242315028-f1aaac82-f01c-4fad-bdbd-2eb4ec2704c3.png](./img/_-ZEKwHeFJ9mpBof/1698242315028-f1aaac82-f01c-4fad-bdbd-2eb4ec2704c3-938033.png)

### 测试
windows5申请管理员证书，公用名Windows3.skills.lan，dns为Windows3.skills.lan

![1714221675124-61d831f7-176b-4014-9ab7-41bf53edf0f7.png](./img/_-ZEKwHeFJ9mpBof/1714221675124-61d831f7-176b-4014-9ab7-41bf53edf0f7-312257.png)

使用IE浏览器访问

![1714273579285-b4598875-404a-4c60-ab7a-123f39847549.png](./img/_-ZEKwHeFJ9mpBof/1714273579285-b4598875-404a-4c60-ab7a-123f39847549-943429.png)

域名正常访问，IP无法访问

![1714273714350-1c3a821b-8b46-41af-811e-a8c2cd25242a.png](./img/_-ZEKwHeFJ9mpBof/1714273714350-1c3a821b-8b46-41af-811e-a8c2cd25242a-515643.png)



> 更新: 2024-04-29 08:45:44  
> 原文: <https://www.yuque.com/gengmouren-1f9qn/whktvz/pccwbzugsc9cd0ei>