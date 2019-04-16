---
layout:     post
title:      Shadowsocks-Linux
subtitle:   
date:       2019-04-15
author:     gary
header-img: 
catalog: true
tags:
    - Linux
---

```
apt-get install python-pip
pip install shadowsocks

vi /etc/shadowsocks.json
{
  "server":"my_server_ip",
  "local_address": "127.0.0.1",
  "local_port":1080,
  "server_port":my_server_port,
  "password":"my_password",
  "timeout":300,
  "method":"aes-256-cfb"
}

#前端启动：
sslocal -c /etc/shadowsocks.json

#后端启动：
sslocal -c /etc/shadowsocks.json -d start

#后端停止：
sslocal -c /etc/shadowsocks.json -d stop

#重启(修改配置要重启才生效)：
sslocal -c /etc/shadowsocks.json -d restart

```

# 解决undefined symbol: EVP_CIPHER_CTX_cleanup
openssl1.1.0已废弃此函数.

vim /usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py
replace libcrypto.EVP_CIPHER_CTX_cleanup to libcrypto.EVP_CIPHER_CTX_reset in two placeses
by https://blog.csdn.net/blackfrog_unique/article/details/60320737


# 开机自启
```
vim /etc/systemd/system/shadowsocks.service

[Unit]
Description=Shadowsocks Client Service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/sslocal -c /etc/shadowsocks.json

[Install]
WantedBy=multi-user.target


systemctl enable /etc/systemd/system/shadowsocks.service
```
