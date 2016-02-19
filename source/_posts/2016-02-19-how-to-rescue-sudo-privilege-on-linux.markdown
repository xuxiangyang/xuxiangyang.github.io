---
layout: post
title:  一次运维失误，误删deployer sudo权限
date: 2016-02-19 16:09:09 +0800
comments: true
categories: 运维
---

经验：
不要用usermod来为用户增加组。比较危险

```
sudo usermod -a -G group user #这是增加
sudo usermod -g group user #这是直接将user的组设置为group
```

用`gpasswd`替代会更安全

```
sudo `gpasswd -a user group #增加
sudo gpasswd -d user group #移除`
```

最后一个sudo 用户的sudo权限消失了怎么办？
两个方案，都需要停服

## 方案一，利用服务器自己的recovery mode

再重启是，上下选择启动项。
{% img /images/posts/20160219/1.png %}
有可能这个recovery mode 在 advance option里。选择 recovery mode
{% img /images/posts/20160219/2.png %}
选root那个，使用root登录
但这个root是一个readonly的系统，需要重新挂在文件系统

```
mount -rw -o remount /
```

然后你就

```
sudo `gpasswd -a user group`
```

就好了

## 方案二，利用linux live mode 挂载硬盘，直接修改文件为用户增加sudo权限

1.制作一个ubuntu desktop的u盘启动盘
2.服务器用u盘启动
3.预览ubuntu（bu an z）
4.执行fdisk找到硬盘，挂载（假设找到了sda1）

```
sudo fdisk -l
sudo mkdir /mnt/root
sudo mount /dev/sda1 /mnt/root
```




5.去修改文件 /etc/sudoers
里面加一行

```
deployer ALL=(ALL:ALL) ALL
```

这个是说给deployer增加sudo权限。以前的做法是把deployer加到sudo这个组里。如果没有洁癖，到这里也可以了。
6.重启服务器，试试`sudo ls`好不好试
7.好使的话

```
sudo `gpasswd -a user group`
```

8.移掉/etc/sudoers
9. 重新登录，看看是不是好使了
