
# GRUB 0.97介绍以及引导修复
***
#0.介绍
***
###基本的情况
  
 * grub 0.97 又称为GRUB Legacy
 * 目前CentOS，Gentoo仍然使用GRUB
 * 配置文件为grub.conf menu.lst
 * 注意(hdX,Y)是第X块硬盘，第Y个分区;X,Y均从0开始计数

###几个阶段

 * 1、装载基本的引导装载程序。基本引导装载程序必须是位于主引导扇区中一个非常小的空间，少于512字节。因此，基本引导装载程序所做的唯一的事情就是装载第二引导装载程序。这主要是归结于在主引导扇区中没有足够的空间用于其他东西了。 
 * 2、装载第二引导装载程序。这第二引导装载程序实际上是引出更高级的功能　，以允许用户装载入一个特定的操作系统。在GRUB中，这步是让用户显示一个菜单或是输入命令。
 * 3、装载在一个特定分区上的操作系统，如linux内核。一旦GRUB从它的命令行或是配置文件中，接到开始操作系统的正确指令，它就寻找必要的引导文件，然后把机器的控制权移交给操作系统。

***
#1.在shell中将grub安装到U盘上
***
```
$ mount /dev/sdb1 /media
$ grub-install  --root-directory=/media/  /dev/sdb
```
必须制定root-directory不然没有boot文件夹
这些完成后，会发现有了boot文件夹，但是此时由于没有修改U盘的MBR，
并不能启动到GRUB
然后还要以下步骤将GRUB安装到MBR上
```
$ grub
grub>find  /boot/grub/stage1
```
会有一个列表，例如：
(hd0,5)
(hd1,0)
grub>root (hd1,0)
root是指的boot目录所在的分区
grub> setup (hd1)
grub>quit

***
#2.修复引导
***
需要处理的有两种情况
* 没有了GRUB(被windows引导覆盖之类的)
* 有GRUB但是无法进入系统(GRUB的配置文件错误)

工具也是两种
* LiveCD
* 安装有GRUB的U盘

***
###(1)没有了GRUB，这时应该安装GRUB到本机硬盘MBR
***
####1)使用安装有GRUB的U盘

使用前面提到的方法制作一个安装有GRUB的U盘

使用该U盘启动后，在grub命令行下修复引导

如果不知道那个分区，使用下面的搜索命令
```
grub>find   /boot/grub/stage1
grub>root  (hd1,0)
此时，hd0是你的U盘，hd1是你的电脑

grub>setup (hd1)
这个会安装到root指定的分区中
```
####2).LiveCD启动后，在shell中修复

shell中修复就是安装GRUB到自己硬盘的MBR

```
$ grub
grub>find  /boot/grub/stage1
会有一个列表，例如：
(hd0,5)
(hd1,0)
grub>root (hd0,5)
root是指的boot目录所在的分区
grub> setup (hd0)
grub>quit
```
***
###(2)有GRUB但是无法进入Linux系统
***
这个时候就要自己使用grub的命令来引导了
感觉命令太复杂，自己记不住

####1)切换到GRUB的命令行，使用configfile
```sh
/boot和/是用一个分区：
grub>find   /boot/grub/stage1
grub>configfile  (hd1,0)/boot/grub/grub.conf
/boot和/不是用一个分区：
grub>find   /grub/stage1
grub>configfile  (hd1,0)/grub/grub.conf
```

这个时候会报错，那么，就需要修改一下引导命令就可以了
####2)使用LiveCD的话
使用Vim修改主机上的配置文件，尝试启动
