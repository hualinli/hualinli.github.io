---
title: 使用Frp配置内网穿透
published: 2025-02-15
description: ''
image: ''
tags: [Network transparency]
category: 'Technology Tutorials'
draft: false 
lang: ''
---

# 使用Frp配置内网穿透

1. **购买云服务器**  
   - 确保选择支持内网穿透的云服务器类型。  
   - 完成注册并登录云服务器控制台。

2. **下载FRP客户端和服务器端**  
   - 访问GitHub下载FRP的最新版本：[FRP-GitHub](https://github.com/fatedier/frp)  
   - 依次下载`frps`（服务器端）和`frpc`（客户端）  
   - 解压`frps`到`/usr/local/frp`目录，解压`frpc`到`/usr/local/frp`目录  
   - 删除不需要的二进制文件

3. **配置服务器端FRP**  
   - 打开`frps.toml`文件并根据需求配置：  
   ```toml
   # /usr/local/frp/frps.toml
   bindPort = 7000  
   transport.tls.force = true  
   auth.token = "public"  

   webServer.addr = "0.0.0.0"  
   webServer.port = 7500  
   webServer.user = " "  
   webServer.password = " "  
   ```
   - 保存并退出编辑器

4. **配置客户端FRP**  
   - 打开`frpc.toml`文件并根据需求配置：  
   ```toml
   # /usr/local/frp/frpc.toml
   serverAddr = ""  
   serverPort = 7000  
   auth.token = "public"  

   [[proxies]]  
   name = "ssh_A"  
   type = "tcp"  
   localIP = "127.0.0.1"  
   localPort = 22  
   remotePort = 6001  

   [[proxies]]  
   name = "web"  
   type = "tcp"  
   localIP = "192.168.1.1"  
   localPort = 80  
   remotePort = 6003  

   [[proxies]]  
   name = "ssh_B"  
   type = "tcp"  
   localIP = "192.168.1.5"  
   localPort = 22  
   remotePort = 6002  
   ```
   - 保存并退出编辑器

5. **设置开机自启（以系统级服务管理为例）**  
   - 打开文件编辑器，创建新的 systemd 服务文件：  
   ```bash
   sudo nano /etc/systemd/system/frp_client.service
   ```
   - 添加以下内容：  
   ```systemd
   [Unit]  
   Description = FRP客户端服务  
   After = network.target syslog.target  
   Wants = network.target  

   [Service]  
   Type = simple  
   ExecStart = /usr/local/frp_0.61.1/frpc -c /usr/local/frp_0.61.1/frpc.toml  
   Restart = always  

   [Install]  
   WantedBy = multi-user.target
   ```
   - 保存文件并退出编辑器  
   - 使服务生效：  
   ```bash
   sudo systemctl daemon-reload  
   sudo systemctl enable frp_client.service  
   sudo systemctl start frp_client.service  
   ```
6. 在云服务器的控制台上打开用到的所有接口。