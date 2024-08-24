---
title: certbot自动化申请ssl证书
author: fjk
date: 2024-08-25 06:12:06
categories:
  - [vpv]
  - [nginx]
  - [ssl]
tags:
  - ssl
  - certbot
  - 证书
---

- 操作系统:``` debian ```

- 安装工具:```snap```

- 步骤

  - 安装snap <!--已安装可跳过-->

    ```bash
    sudo apt update
    sudo apt install snapd
    ```

  - 利用snap安装certbot

    ```bash
    sudo snap install --classic certbot
    ```

  - 创建软连接

    ```bash
    sudo ln -s /snap/bin/certbot /usr/bin/certbot
    ```

  - 证书申请

    - 单域名证书申请

      - 申请证书后自动为你修改nginx配置完成ssl证书配置

        ```bash
        sudo certbot --nginx
        ```

      - 只申请nginx下配置的站点的证书，不修改nginx配置文件，手动部署证书

        ```bash
        sudo certbot certonly --nginx
        ```

    - 利用域名dns申请泛域名证书

      - 获取dns服务商密钥

      - 将cloudflare密钥保存到服务器

        ```bash
        snap set certbot trust-plugin-with-root=ok
        touch ~/.secret/certbot/cloudflare.ini
        ```
      
        - 如果使用的是全局密钥<!--Global API Key -->使用下方的配置
      
          ```ini
          #dns_cloudflare_email = 邮箱地址 如果用的是GlobalApi需要
          dns_cloudflare_api_key = 全局密钥
          ```
      
        - 如果使用的是非全局密钥，使用下方的配置
      
          ```ini
          dns_cloudflare_api_token = 非全局密钥
          ```
      
      - 配置完密钥，修改密钥文件的权限，否则可能会报错
      
        ```bash
        chmod 400 ~/.secret/certbot/cloudflare.ini
        ```
      - 安装certbot-dns-cloudflare插件
        ```bash
        snap install certbot-dns-cloudflare
        ```
      - 开始申请证书
        ```bash
        certbot certonly \
        --dns-cloudflare \
        --dns-cloudflare-credentials ~/.secret/certbot/cloudflare.ini \
        --dns-cloudflare-propagation-seconds 60 \
        -d cloudflare.icu \
        -d *.cloudflare.icu
        ```
      
        <!-- 请把上方命令替换成自己需要申请泛域名证书的域名 -->
      
      - 申请证书之后，可以用```sudo certbot renew --dry-run```命令验证证书申请情况
