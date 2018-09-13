---
title: 关于 SSH 端口转发
date: 2018-09-12 20:32:47
tags:
    - ssh
    - Linux
categories: 技术
---

SSH有三种端口转发模式
* 本地端口转发(Local Port Forwarding)
* 远程端口转发(Remote Port Forwarding)
* 动态端口转发(Dynamic Port Forwarding)

对于本地/远程端口转发，两者的方向恰好相反。动态端口转发则可以用于科学上网。

SSH端口转发也被称作SSH隧道(SSH Tunnel)，因为它们都是通过SSH登陆之后，在SSH客户端与SSH服务端之间建立了一个隧道，从而进行通信。SSH隧道是非常安全的，因为SSH是通过加密传输数据的(SSH全称为Secure Shell)。

常见的用途就是在管理远程数据库时，开启远程登陆是非常不安全的。所以通过隧道将其端口转发到本地（例如：mysql 的3306，redis的 6379）

<!--more-->


### 本地端口转发

所谓本地端口转发，就是将发送到**本地端口**的请求，转发到**目标端口**。这样，就可以通过访问本地端口，来访问目标端口的服务。
使用-L属性，就可以指定需要转发的端口，语法是这样的:

```bash
ssh [-L 本地地址:本地端口:远程地址:远程端口] [user@]hostname [command]
```

通过本地端口转发，可以将发送到**本地主机**端口的请求，转发到**远程云主机**的某个端口请求。

例如一个远程主机的python代码, 服务端本地开放5000端口

![](https://ww1.sinaimg.cn/large/005YhI8igy1fv76rpaqdvj311w0skn18)

我们通过以下命令来实现本地访问

```bash
ssh -L 5000:localhost:5000 root@xxxxx
```
![](https://ww1.sinaimg.cn/large/005YhI8igy1fv76wiys5lj31760asdiv)

另外，-L选项中的目标地址也可以是其他主机的地址。



### 远程端口转发

所谓远程端口转发，就是将发送到**远程端口**的请求，转发到**目标端口**。这样，就可以通过访问远程端口，来访问目标端口的服务。
使用-R属性，就可以指定需要转发的端口，语法是这样的:

```bash
ssh [-R 远程地址:远程端口:本地地址:本地端口:] [user@]hostname [command]
```

应用场景：
本地主机A1运行了一个服务，端口为3000，远程云主机B1需要访问这个服务。

本次示例就是将上面的倒过来即可，即远程做访问，本地做服务

![](https://ww1.sinaimg.cn/large/005YhI8igy1fv77522huuj311208iq4x)

### 远程端口转发

对于**本地端口转发**和**远程端口转发**，都存在两个一一对应的端口，分别位于SSH的客户端和服务端，而动态端口转发则只是绑定了一个本地端口，而**目标地址:目标端口**则是不固定的。**目标地址:目标端口**是由**发起的请求**决定的，
比如，请求地址为192.168.1.100:3000，则通过SSH转发的请求地址也是192.168.1.100:3000。
常常用于科学上网，代理上网等

语法：

```bash
ssh [-D 本地地址:本地端口] [user@]hostname [command]
```

这时，通过**动态端口转发**，可以将在本地主机发起的请求，转发到远程主机，而由远程主机去真正地发起请求。

例如将本地代理设置为：
![](https://ww1.sinaimg.cn/large/005YhI8igy1fv840hv9irj317c11idnf)

之后执行命令

```bash
ssh -D localhost:1088 root@xxx
```
这是本地的所有的网络请求到将转发到1088端口上之后有ssh通过云主机转发到真正的请求地址，如果你的云主机可以访问外网的话，完全可以达到科学上网的目的。

### 参考链接

* [SSH/OpenSSH/PortForwarding](https://help.ubuntu.com/community/SSH/OpenSSH/PortForwarding?highlight=%28%28SSH%7COpenSSH%7CPortForwarding%29%29)

* [SSH隧道翻墙的原理和实现](http://www.pchou.info/linux/2015/11/01/ssh-tunnel.html)