---
title: 为自建的网站增加https支持
date: 2018-08-01 20:01:19
tags:
- 技术
- 建站
categories:
- 技术
---

# 为自建的网站增加https支持

## 使用acme.sh获取SSL证书

[Let's Encrypt](https://letsencrypt.org/)提供的SSL证书不仅免费，且支持泛域名，是十分便利的选择，唯一的缺点是证书只有60天的有效期。而[acme.sh](https://github.com/Neilpang/acme.sh)实现了acme协议，可以完成从申请到续签的一系列操作，我们使用acme.sh来管理证书。

### 安装acme.sh

```bash
curl  https://get.acme.sh | sh
```

acme.sh会被安装在`~/.acme.sh/`中，并创建cronjob，每天0点检查证书是否将过期并重新申请。

<!-- more -->

### 申请证书

我们使用dns方式，而acme.sh还提供[另一种http方式](https://github.com/Neilpang/acme.sh/wiki/%E8%AF%B4%E6%98%8E#2-%E7%94%9F%E6%88%90%E8%AF%81%E4%B9%A6)。
先获取解析商提供的api id，以阿里云为例，[获取api id](https://ak-console.aliyun.com/#/accesskey)。
其他解析商可以看[官方文档](https://github.com/Neilpang/acme.sh/blob/master/dnsapi/README.md)。

{% asset_image aliyun.png 阿里云解析 %}

```bash
export Ali_Key="AccessKey ID"
export Ali_Secret="Access Key Secret"
```

Acme.sh会自动记录api id，续期时无需再次输入。

```bash
acme.sh --issue --dns dns_ali -d example.com #单域名
acme.sh --issue --dns dns_ali -d example.com -d blog.example.com #多域名
acme.sh --issue --dns dns_ali -d *.example.com #泛域名
```

证书会出现在`~/.acme.sh/`。

## 为Apache2配置https支持

以Apache2为例，Nginx类似。

首先确保安装了`opensll`，并开启Apache2的SSL模块。

```bash
sudo apt install openssl
sudo a2enmod ssl
```

Apache2的站点配置文件在`/etc/apache2/sites-enabled/`下，通常存在一个`000-default.conf`。将其复制一份修改为`001-ssl.conf`开启https。

```bash
sudo cp /etc/apache2/sites-enabled/000-defalut.conf /etc/apache2/sites-enabled/001-ssl.conf
sudo vim /etc/apache2/sites-enabled/001-ssl.conf
```

编辑`001-ssl.conf`

- 将`<VirtualHost *:80>`改为`<VirtualHost *:443>`
- 添加如下内容

```bash
SSLEngine On
SSLOptions +StrictRequire
SSLCertificateFile /etc/ssl/certs/cert.cer #证书文件
SSLCertificateKeyFile /etc/ssl/private/key.key #key文件
SSLCertificateChainFile /etc/apache2/ssl.crt/fullchain.cer #fullchain
```

注意创建文件夹和权限的问题。可以将`~/.acme.sh/`下的证书手动复制测试一下Apache2是否配置成功。

## 使用acme.sh自动续期证书

```bash
acme.sh --install-cert -d example.com \
--cert-file      /etc/ssl/certs/cert.cer \ #acme.sh给的3个证书文件在Apache2配置中的位置
--key-file       /etc/ssl/private/key.key \
--fullchain-file /etc/apache2/ssl.crt/fullchain.cer \
--reloadcmd     "service apache2 force-reload" #让Apache2重新加载SSL证书的命令
```

分别告诉acme.sh个文件应该放置的位置和让Apache2重新加载SSL证书的命令。

acme.sh会记住这些命令，以自动续期证书。

## 为Apache2配置https强制跳转

将所有http请求强制转换为https请求，以增强安全性。

打开`/etc/apache2/sites-enabled/000-default.conf`，在`<VirtualHost *:80> ... </VirtualHost>`标签中插入下面的内容。

```bash
RewriteEngine on
RewriteCond  %{HTTPS} !=on
RewriteRule  ^(.*) https://%{SERVER_NAME}$1 [L,R]
```

大功告成。
