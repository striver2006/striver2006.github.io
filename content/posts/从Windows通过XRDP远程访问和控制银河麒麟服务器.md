+++
date = '2025-04-19T16:07:00+08:00'
draft = false
title = '从Windows通过XRDP远程访问和控制银河麒麟服务器'
tags = ["银河麒麟", "XRDP"]
+++
### 1.服务器安装XRDP
银河麒麟服务器版的官方yum源并没有引入xrdp软件，所以无法直接通过yum方式安装。我们可以通过下载rpm软件包，
然后rpm方式安装。访问xrdp官网[https://rhel.pkgs.org/8/epel-x86_64/xrdp-0.10.1-1.el8.x86_64.rpm.html](https://rhel.pkgs.org/8/epel-x86_64/xrdp-0.10.1-1.el8.x86_64.rpm.html)
找到对应的版本软件包，银河麒麟服务器版v10是基于centos8的，下载xrdp-0.10.1-1.el8.x86_64.rpm即可。

### 2.下载xrdp软件的rpm包
```shell
wget https://dl.fedoraproject.org/pub/epel/8/Everything/x86_64/Packages/x/xrdp-0.10.1-1.el8.x86_64.rpm
```

### 3.rpm安装软件包
```shell
rpm -Uvh xrdp-0.10.1-1.el8.x86_64.rpm
```
在终端中执行sudo vim /usr/libexec/xrdp/startwm.sh
在最后添加如下内容：
unset DBUS_SESSION_BUS_ADDRESS
unset XDG_RUNTIME_DIR

### 4.启动服务后加入开机自启同时开放3389端口
```shell
systemctl start xrdp
systemctl enable xrdp
firewall-cmd --zone=public --add-port=3389/tcp --permanent
firewall-cmd --reload
```

### 5.远程桌面连接服务器版
5.1.在Windows系统里按Win+R键，并且输入mstsc回车或点击【确定】按钮或在开始里找到远程桌面连接，点击打开。  

5.2.在弹出窗口中输入服务器的IP地址，回车或点击【连接】按钮。  

5.3.Session选择"Xvnc"，输入服务器上的用户名和密码，回车或点击 OK 按钮。  

5.4.进入服务器远程桌面。  

注：在银河麒麟高级服务器操作系统上安装并启动xrdp服务后，登录成功但遇到了“could not acquire name on session bus”的错误提示，导致无法正常访问桌面。
![错误提示](../images/从Windows通过XRDP远程访问和控制银河麒麟服务器/1.png "错误提示")
<div style="font-size:14px;color:#C0C0C0; text-align: left;">图1 错误提示</div>

【问题分析】  

该问题与环境变量DBUS_SESSION_BUS_ADDRESS的设置有关。
  

【问题解决方法】  

1、修改/etc/sysconfig/desktop文件：  
```shell
vim  /etc/sysconfig/desktop
```
在DESKTOP变量前添加一行unset DBUS_SESSION_BUS_ADDRESS内容后，保存并关闭文件。
![修改配置文件](../images/从Windows通过XRDP远程访问和控制银河麒麟服务器/2.png "修改配置文件")
<div style="font-size:14px;color:#C0C0C0; text-align: left;">图2 修改配置文件</div>  

修改完成后重启 xrdp-sesman 服务与 xrdp 服务即可生效。  
2、重启xrdp-sesman及xrdp服务：
```shell
systemctl restart xrdp-sesman
systemctl restart xrdp
```

3、验证问题是否解决：  

尝试重新连接xrdp会话。  

