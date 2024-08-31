---
title: 在Linux上搭建mihomo(clash.meta)服务
author: fjk
categories:
  - - default
tags:
  - default
date: 2024-08-31 19:29:27
---

Clash Meta 现已改名为 mihomo。本文为 Debian 12 虚拟机上搭建 mihomo 服务的过程，供有需要的人士参考。

## 一、下载安装 mihomo 服务

在官方下载页面下载适合自己 Linux 系统的二进制安装包，比如 mihomo-linux-amd64-alpha-b3db113.gz，解压后将 mihomo-linux-amd64 上传到虚拟机上，同时重命名为 mihomo：

```
scp /path/mihomo-linux-amd64 user@host:~/mihomo
```

SSH 登录进主机，给 mihomo 增加执行权限：

```
sudo chmod +x mihomo
```

将 mihomo 移动到 `/usr/local/bin/` 目录：

```
sudo cp mihomo /usr/local/bin
```

创建运行目录：

```
sudo mkdir /etc/mihomo -p
```

创建配置文件：

```
sudo vi /etc/mihomo/config.yaml
```

在其中添加你的配置内容，因为 mihomo 兼容 clash 配置，所以直接将可用的 clash 配置复制粘贴进去即可。

在将 mihomo 添加到系统服务之前，最好手动运行一次观察是否正常。

```
sudo /usr/local/bin/mihomo -d /etc/mihomo
```

如果启动成功，没出现错误信息，那就可以创建 mihomo 服务：

```
sudo vi /etc/systemd/system/mihomo.service
```

粘贴以下内容：

```
[Unit]
Description=mihomo Daemon, Another Clash Kernel.
After=network.target NetworkManager.service systemd-networkd.service iwd.service

[Service]
Type=simple
LimitNPROC=500
LimitNOFILE=1000000
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_RAW CAP_NET_BIND_SERVICE CAP_SYS_TIME CAP_SYS_PTRACE CAP_DAC_READ_SEARCH
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_RAW CAP_NET_BIND_SERVICE CAP_SYS_TIME CAP_SYS_PTRACE CAP_DAC_READ_SEARCH
Restart=always
ExecStartPre=/usr/bin/sleep 1s
ExecStart=/usr/local/bin/mihomo -d /etc/mihomo
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

说明：vi 可通过按 ESC，再输入 `:wq` 保存退出。

重载 systemd：

```
systemctl daemon-reload
```

启用 mihomo 服务（开机、重启系统后自动启动）：

```
systemctl enable mihomo
```

立即启动 mihomo 服务：

```
systemctl start mihomo
```

检查 mihomo 服务状态：

```
systemctl status mihomo
```

没问题的话就已经安装好 mihomo 服务了。

## 二、添加 Web 控制面板

刚才部署好了 mihomo 服务，如果要切换节点、重载配置等都需要 ssh 登录到虚拟机上修改，较为不便。所以我们还需要有控制面板。如果你是 Linux 桌面用户，可以用一些客户端很方便。但这次是在不带桌面的虚拟机中（相当于 Linux VPS）部署，所以需要安装一个 Web 控制面板进行简单的管理。

通过 Web 控制面板管理，需要在配置文件中添加以下内容：

```
external-controller: 0.0.0.0:9090
```

注意：0.0.0.0 意味着可以在公网访问，如果你的机器是直接暴露在公网的，请勿如此配置。

### 1. 通过第三方部署的 Web 控制面板管理

一般来说，我们可以通过部署在外部的 yacd 或 metacubexd 等面板管理，这是最方便的形式。如果遇到了 failed to connect 错误，是因为没有允许在 https 网站加载 http 内容。解决方式只要对第三方部署的面板设置允许不安全内容即可。如下图：

![metacubexd 网站设置允许不安全内容](https://librecat.me/wp-content/uploads/2024/03/17107578582.webp)metacubexd 网站设置允许不安全内容

![metacubexd 网站设置允许不安全内容](https://librecat.me/wp-content/uploads/2024/03/17107578581.webp)metacubexd 网站设置允许不安全内容

### 2. 自行部署 Web 控制面板

另外一种方式是自行部署。mihomo 支持在配置文件中添加 `external-ui` 来指定 Web 控制器的目录，这样我们就不需要再安装一个 nginx 来提供 Web 服务，比较省事。

注意：ClashForWindows 等控制器不支持 `external-ui` 配置，添加后会报错。

例如我想将控制器放在 /etc/mihomo/ui 中，则添加如下配置：

```
external-ui: /etc/mihomo/ui
```

从 GitHub 下载：

```
sudo git clone https://github.com/metacubex/metacubexd.git -b gh-pages /etc/mihomo/ui
```

重启 mihomo 服务

```
sudo systemctl restart mihomo
```

现在就可以在浏览器中访问 Web 控制器了。有人会遇到访问不了的情况，一般是因为浏览器地址栏的路径填写错误，正确的地址应该是 `http://ip:port/ui`，例如：

```
http://192.168.1.202:9090/ui
```

## 三、定时从网络更新配置文件

如果你是用的机场的配置文件，如果机场变更了配置，你还需要手动更新，比较麻烦。还好我们可以通过定时任务来自动更新配置文件。

```
sudo crontab -e
```

如果你第一次使用 crontab，会让你选择编辑器，我选择 VI。进入定时任务配置文件，按 i 进入编辑状态，新起一行（可以是文件头部）输入：

```
0 4 * * * curl -L -o /etc/mihomo/config.yaml -A "clash" SUB_URL; systemctl reload mihomo
```

输入 : ，再输入 wq，按 回车 即可保存。

以上配置的意思是：在每天夜里 4 点从 SUB_URL 更新订阅配置，配置保存在 /etc/mihomo/config 中，更新完成后重载 mihomo 服务。记得自行替换 SUB_URL 为机场给到的订阅地址。
