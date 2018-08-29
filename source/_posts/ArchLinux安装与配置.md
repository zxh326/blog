---
title: ArchLinux安装与配置
date: 2017-05-20 22:54:24
tags: [Arch,linux]
categories:
    - linux
    - arch
---

> A simple, lightweight distribution
> 一个简单，轻便的Linux操作系统，
> Archlinux有一个强大的wiki，本文依据[官方wiki](https://wiki.archlinux.org/)编写
> 如有不当之处，请指出！

<!-- more -->

# 准备和磁盘分区

## 准备
*   硬件
    *   一个大于等于4GB的U盘
    *   电脑（很明显）
*   软件
    *   [Arch linux ISO](https://www.archlinux.org/download/)镜像
    *   如果你对速度有要求的话强烈建议从国内源下载
        * [清华大学源 TUNA](http://mirrors.163.com/archlinux/iso/)
        * [中国科学技术大学 USTC](https://mirrors.ustc.edu.cn/archlinux/iso/)
        * [大连东软 Neusoft](https://mirrors.neusoft.edu.cn/archlinux/iso/)
        * [这里有所有China源](https://www.archlinux.org/mirrorlist/?country=CN&protocol=http&protocol=https&ip_version=4)
## 制作启动U盘
*   在Windows下制作可启动U盘
    *   [Rufus](https://rufus.akeo.ie)
    *   用法这里不多说
*   在类Unix环境下制作可启动U盘
    *   使用底层dd命令
        * Linux :
            * `lsblk`命令列出磁盘
        * Mac下
            * `diskutil list`命令
    *   之后执行
        * `sudo dd if=/path_to_arch_.iso of=/dev/sdX`

## 在BIOS中启动EFI模式

   在**BIOS**里找到**UEFI/EFI mode**并且选中即可。有些BIOS可能有什么**Fast boot和safe mode**什么的，需要把这两个给**disable**

## 在U盘中启动

在启动的时候选择从**U盘中**启动
普通笔记本一般都是F2,F10,F12之类的


## 检查互联网连接
```bash
    ping -c 3 www.baidu.com
```
如果出现
```bash
$ ping -c 3 www.baidu.com
PING baidu.com (123.125.114.144) 56(84) bytes of data.
64 bytes from 123.125.114.144 (123.125.114.144): icmp_seq=1 ttl=57 time=5.19 ms
64 bytes from 123.125.114.144 (123.125.114.144): icmp_seq=2 ttl=57 time=5.22 ms
64 bytes from 123.125.114.144 (123.125.114.144): icmp_seq=3 ttl=57 time=5.22 ms

--- baidu.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 5.196/5.215/5.227/0.060 ms
```
Success ! 
如果出现
```bash
$ ping -c 3 www.baidu.com
ping: cannot resolve www.baidu.com: Unknown host
```
那么就说明你没有网络。

如果你没有网络的话，检查有没有叉网线。如果没有网线的话，输入`wifi-menu -o`以连接`Wi-Fi`。

## 检查是否启动EFI模式

输入

```bash 
efivar -l
```

如果出现一大堆东西，那么就说明你在使用`UEFI`模式。

## 磁盘分区
*   找到所有磁盘
    *   输入命令`lsblk`。可能会出现`/dev/sda，/dev/sdb，/dev/sdc`什么的。找到你要安装`Linux`的磁盘。本文一下将使用`/dev/sdX`作为例子，请根据**实际需要**做出**适当更改**。  
*   清除已存在分区
    *   输入`gdisk /dev/sdX`然后依次输入`x，z，y`和`y`。请务必小心，这个命令会把`/dev/sdX`**整个磁盘**给**抹掉**。**啪啪啪啪～**

### 创建分区

*   输入命令`cgdisk /dev/sdX`。会出现`Any key to continue`（随便摁个键继续）字样。

####   创建boot分区

> 找到`free space`然后把光标移动到**[New]**然后回车

```bash
First Sector:                       # 直接回车
Size in sectors: 100MiB             # 回车
Hex Code: EF00                      # 回车
Enter new partition name: boot      # 回车
```
####   创建Swap分区

>  找到free space然后把光标移动到**[New]**然后回车。本文以2GiB大小的swap分区为例子。请根据实际情况自行调整。

```bash
First Sector:                       # 直接回车
Size in sectors:                    # 回车
Hex Code: 8200                      # 回车
Enter new partition name: swap      # 回车
```
####   创建root分区

> 找到free space然后把光标移动到**[New]**

```bash
First Sector:                       # 直接回车
Size in sectors:                    # 回车 
Hex Code:                           # 回车 
Enter new partition name: root      # 回车                 
```
之后把光标移动到**[Write]**回车再输入**yes**。最后，光标移动到**[Quit]**回车。

###   选择file systems

>  本文使用ext4，根据实际情况可以**酌情处理**（切记**不要**直接复制我的**回车**）。 依次输入命令:

```bash
mkfs.fat -F32 /dev/sda1             # 你的boot分区
mkswap /dev/sda2                    # 你的swap分区
swapon /dev/sda2                    # 同上
mkfs.ext4 /dev/sda3                 # 你的root分区
```

现在最好重启计算机，输入reboot。

# 安装Arch Linux并且让它能开机

## Mounting Partitions

输入:

```bash
mount /dev/sda3 /mnt
mkdir /mnt/boot
mkdir /mnt/home                     # 仅对创建Home分区的需要这一步
mount /dev/sda1 /mnt/boot
```

**!如果你是在windows下安装Arch需要双系统的话 建议执行这一步**
```bash
mkdir /mnt/boot/efi
mount /dev/你的windows引导分区 /mnt/efi
```

## 设置Arch Repository Mirrorlist:
* 创建一个备份

```bash
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
```

* 获取与你链接最快的源 两种方法
    * 第一种将全世界所有源测速，来判断哪一个更适用你**此方法较慢**
    `rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist`
    * 第二种在[Arch wiki 官方](https://www.archlinux.org/mirrorlist/)选定特定的国家
    * 第三种 自己在`/etc/pacman.d/mirrorlist`一直dd删除你不想要的源

## 安装Arch Base文件
输入 `pacstrap -i /mnt base base-devel`
整个过程一直摁回车和y即可，速度依据你上一步选择的源的速度

## 生成Fstab文件
输入 `genfstab -U -p /mnt >> /mnt/etc/fstab`

查看 `fastab` 文件是否正确
输入 `vim /mnt/etc/fstab`
应该看到类似于
```
/dev/sda2 none swap defaults 0 0
或者
UUID=xxxxxxxxxxxxxxxxxxxxx none swap defaults 0 0
```
如果是这样 保存退出即可，如果不是，检查安装有没有出错，重新来即可

如果一切顺利的话就可以进入我们的新安装的系统啦
`arch-chroot /mnt`
嗯 到这里我们就成功一大截了。

## 本地化
* 语言
    * 输入 `vim /etc/locale.gen`
    * 找到 `en_US.UTF-8 UTF-8`,`zh_CN.UTF-8`
    * 将其前面的`#`注释去掉保存即可
    * 现在输入`locale-gen`产生locale
    * 最后输入 `echo LANG=en_US.UTF-8 > /etc/locale.conf`
    * `export LANG=en_US.UTF-8`
    * 语言化完成
* 时区
**本文以上海时区为例子**
    *  `ln -s /usr/share/zoneinfo/Asia/Shanghai > /etc/localtime`
    * 设置硬件时钟
        * `hwclock --systohc --utc`

> TIPS
> 如果双系统发现时钟与Linux时钟不一样，原因是windows默认时区问题
> 这里不过多阐述
> 可以修改注册表，让windows将硬件时钟当作标准时间
>　Reg add HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1
> 也可以参考网上其他办法





