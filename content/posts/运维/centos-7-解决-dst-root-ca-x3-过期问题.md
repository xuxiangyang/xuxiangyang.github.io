+++
title = "Centos 7 解决 DST Root CA X3 过期问题"
author = ["徐向阳"]
date = 2021-10-08T14:27:00+08:00
tags = ["运维"]
categories = ["运维"]
draft = false
+++

[2021年9月30日 Let's Encrypt的DST Root CA X3 证书过期了](https://letsencrypt.org/docs/dst-root-ca-x3-expiration-september-2021/)。
给我的实际影响是 使用 <https://gems.ruby-china.com> 没办法安装gem，会报ssl错误。
如果你是openssl 1.1.0 不会遇到这个问题，但centos 7 (我的版本是7.3.1611) 默认安装的是 openssl 1.0.2。
如果机器少，你可以根据 openssl 的[博客](https://www.openssl.org/blog/blog/2021/09/13/LetsEncryptRootCertExpire/) 手动修改证书。
但我的机器多，手动修改证书也很怕出错，所以可以执行下面的命令

```bash
yum update -y ca-certificates && update-ca-trust
```

至少需要升级到 `2021.2.50-72` 版本，其实原理是一样的，只是[ca-certificates 在2020-06-09的版本移除了DST Root CA X3](https://centos.pkgs.org/7/centos-updates-x86%5F64/ca-certificates-2021.2.50-72.el7%5F9.noarch.rpm.html)
