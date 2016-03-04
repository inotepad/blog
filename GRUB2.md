
#GRUB2 介绍以及引导修复
***
##0.介绍
***
* http://www.gnu.org/software/grub/manual/grub.html
* https://help.ubuntu.com/community/Grub2
* 注意(hdX,Y)是第X块硬盘，第Y个分区 X0开始计数,Y均从1开始计数 
* 配置文件为grub.cfg
* 强大的loopback详见grub2安装系统
* grub-mkconfig可以自动生成配置文件
* 可以定义变量 grub中 root (hd0,0) grub2 set root=(hd0,1)
* 安装程序grub和安装grub到mbr是不一样的
* GRUB2中的root就是/boot中的/所在的分区
***
##1.GRUB2 安装到U盘
***
```
$ grub-install --root-directory=/mnt/ /dev/sdb
```
GRUB2下是不可以只在grub中修复引导的 这是相对与grub不足的地方

***
##2.使用GRUB中修复引导
***

需要处理的有两种情况

* 没有了GRUB(被windows引导覆盖之类的)
* 有GRUB但是无法进入系统(GRUB的配置文件错误)

工具也是两种

* LiveCD
* 安装有GRUB的U盘

*** 
###(1)GRUB被覆盖
***
#####1)使用LiveCD修复

第一种方法

######直接使用LiveCD中的GRUB安装到本机硬盘上
```
如果boot不是单独的分区，就是所/和/boot是同一个分区
$ mount /dev/sda2 /mnt$ grub-install --root-directory /mnt /dev/sda
grub会自动安装到boot文件夹中
如果boot是单独一个分区$ mount /dev/sda2 /mnt
$ mount /dev/sda1 /mnt/boot$ grub-install --root-directory /mnt /dev/sda
```

第二种方法
######切换到本机硬盘上，在安装
```
####1.Disk unity,找到fedora安装所在分区
fdisk -l也可以
####2.mount -t ext4 -o rw /dev/sdaX /media
####3.mount -t proc none /media/proc
mount -o bind /dev /media/dev
mount -o bind /sys /media/sys
mount -t devpts devpts /media/dev/pts
####4.grub2安装到硬盘MBR上恢复引导chroot /media
grub2-install /dev/sda
####5.安装grub2grub2-mkconfig  -o  /boot/grub2/grub.cfg
reboot####6.exit
umount /media/proc
umount /media/dev/pts
umount /media/sys
umount /media/dev
umount /media
reboot
```

#####2) 可以或者是使用安装有GRUB的U盘，然后通过configfile进入系统
 ```
grub>cat (hd1,0)
可以查看每个分区中的文件，来确定自己要使用的分区是那个
grub>configfile (hd1,2)/boot/grub/grub.cfg
可能需要修改一些参数
这样就可以进入系统了
$ grub-install /dev/sda
这样是安装到MBR
```
****
###(2)GRUB有，但是不能进入系统
****
#####1)使用LiveCD修复
使用Vim修改主机上的配置文件，尝试启动
#####2)使用安装有GRUB的U盘修复
自己使用GRUB的命令进入系统，通过configfile进入系统后（中间要修改一些参数）
然后修改GRUB配置文件的参数

