---
title: mihomo配置文件模板
author: fjk
categories:
  - - default
tags:
  - default
date: 2024-08-27 09:30:17
---

```yaml
dns:
  enable: true
  use-system-hosts: true
  listen: 0.0.0.0:53
  ipv6: true
  default-nameserver:
    - 114.114.114.114
    - 223.5.5.5
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
  nameserver:
    - https://dns.alidns.com/dns-query
    - https://doh.pub/dns-query
  fallback:
    - 114.114.114.114
    - 223.5.5.5
 
proxies:
# 下面放入你的节点信息（可以多个，认证仔细点！），节点名字支持 🇭🇰 🇺🇸 🇸🇬 🇲🇴 🇬🇧 🇩🇪 🇯🇵 🇰🇷 🇮🇳 等图标。
- name: inter
  type: socks5
  server: 
  udp: true
  port: 
  username: 
  password: 

- name: intra
  type: socks5
  udp: true
  server: 
  port: 
  username: 
  password: 


- name: 🇸🇬 AWS 🚀
  type: hysteria2
  server: 
  port: 
  password: 
  sni: https://bing.com
  skip-cert-verify: true

- name: 🇸🇬 AWS 🛡️
  type: vless
  server: 
  port: 
  uuid: 
  servername: learn.microsoft.com
  network: tcp
  udp: true
  tls: true
  reality-opts:
    public-key: tsvdWTIkb5BlqErQCf5sct8wnXTF3PDZJQB4wGMOCQw
    short-id: fd3c643b
  client-fingerprint: chrome
  flow: xtls-rprx-vision

- name: 🇭🇰 Akile 💊
  type: hysteria2
  server: 
  port: 443
  password: Cc/tMY0YQBN82AxrdArE5Nyq
  sni: https://bing.com
  skip-cert-verify: true

- name: 🇺🇸 bwghost 🐢
  type: hysteria2
  server: 
  port: 443
  password: Sz82qL41CXp23Y2E3eAlm8ab
  sni: https://news.ycombinator.com/
  skip-cert-verify: true

proxy-groups:
  - name: ♻️ 自动选择
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 300
    tolerance: 50
    proxies:
      - 🇸🇬 AWS 🚀
      - 🇸🇬 AWS 🛡️
      - 🇭🇰 Akile 💊
      - 🇺🇸 bwghost 🐢
  - name: 🚀 节点选择
    type: select
    proxies:
      - ♻️ 自动选择
      - DIRECT
      - 🇸🇬 AWS 🚀
      - 🇸🇬 AWS 🛡️
      - 🇭🇰 Akile 💊
      - 🇺🇸 bwghost 🐢
  - name: 🔰 政务互联网
    type: select
    proxies:
      - DIRECT
      - inter

  - name: ⚖️ 政务外网
    type: select
    proxies:
      - DIRECT
      - intra
rules:
 - IP-CIDR,10.109.0.0/16,🔰 政务互联网,no-resolve
 - IP-CIDR,10.111.0.0/16,🔰 政务互联网,no-resolve

 - IP-CIDR,172.18.0.0/16,⚖️ 政务外网,no-resolve
 - IP-CIDR,172.21.0.0/16,⚖️ 政务外网,no-resolve
 - IP-CIDR,172.26.0.0/16,⚖️ 政务外网,no-resolve
 - IP-CIDR,10.135.0.0/16,⚖️ 政务外网,no-resolve

 - RULE-SET,adblok,REJECT
 - DOMAIN,clash.razord.top,DIRECT
 - DOMAIN,yacd.haishan.me,DIRECT
 - RULE-SET,direct,DIRECT
 - RULE-SET,lan-cidr,DIRECT
 - RULE-SET,cn-cidr,DIRECT
 - RULE-SET,private,DIRECT
 - GEOIP,LAN,DIRECT
 - GEOIP,CN,DIRECT
 - RULE-SET,icloud,🚀 节点选择
 - RULE-SET,apple,🚀 节点选择
 - RULE-SET,google,🚀 节点选择
 - RULE-SET,proxy,🚀 节点选择
 - RULE-SET,gfw,🚀 节点选择
 - RULE-SET,tld-not-cn,🚀 节点选择
 - RULE-SET,telegram-cidr,🚀 节点选择
 - RULE-SET,applications,DIRECT
 - MATCH,🚀 节点选择

rule-providers:
  adblok:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/reject.txt"
    path: ./ruleset/reject.yaml
    interval: 86400

  icloud:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/icloud.txt"
    path: ./ruleset/icloud.yaml
    interval: 86400

  apple:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/apple.txt"
    path: ./ruleset/apple.yaml
    interval: 86400

  google:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/google.txt"
    path: ./ruleset/google.yaml
    interval: 86400

  proxy:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/proxy.txt"
    path: ./ruleset/proxy.yaml
    interval: 86400

  direct:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/direct.txt"
    path: ./ruleset/direct.yaml
    interval: 86400

  private:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/private.txt"
    path: ./ruleset/private.yaml
    interval: 86400

  gfw:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/gfw.txt"
    path: ./ruleset/gfw.yaml
    interval: 86400

  tld-not-cn:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/tld-not-cn.txt"
    path: ./ruleset/tld-not-cn.yaml
    interval: 86400

  telegram-cidr:
    type: http
    behavior: ipcidr
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/telegramcidr.txt"
    path: ./ruleset/telegramcidr.yaml
    interval: 86400

  cn-cidr:
    type: http
    behavior: ipcidr
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/cncidr.txt"
    path: ./ruleset/cncidr.yaml
    interval: 86400

  lan-cidr:
    type: http
    behavior: ipcidr
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/lancidr.txt"
    path: ./ruleset/lancidr.yaml
    interval: 86400

  applications:
    type: http
    behavior: classical
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/applications.txt"
    path: ./ruleset/applications.yaml
    interval: 86400
```

