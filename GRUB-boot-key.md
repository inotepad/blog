# 使用GRUB制作引导多个ISO的U盘
***
##1.制作一个有GRUB2的U盘
```
$ grub-install --root-directory=/mnt/ /dev/sdb
/dev/sdb是你的U盘设备
```
***
##2.配置文件
***
####注意
* 在安装CentOS时，U盘是/dev/sda，电脑中的硬盘是/dev/sdb
* 配置文件/boot/grub/grub.cfg 

####内容如下：
```
#已经完成Ubuntu LiveCD , CentOS DVD , Fedora DVD
#未完成BT5 LiveCD,CentOS LiveCD,Fedora DVD. Debian
set timeout=20
set default=0
menuentry "Ubuntu LiveCD" {
    set isofile="Ubuntu.iso"
    loopback loop (hd0,1)/$isofile
    linux (loop)/casper/vmlinuz boot=casper iso-scan/filename=$isofile noprompt noeject
    initrd (loop)/casper/initrd.lz
}
menuentry "Fedora DVD" {
    set isofile="Fedora-17-i386-DVD.iso"
    loopback loop (hd0,1)/$isofile
    linux (loop)/images/pxeboot/vmlinuz  linux  repo=hd:/dev/sdb1:/
    initrd (loop)/images/pxeboot/initrd.img 
}
menuentry "CentOS DVD" {
    set isofile="CentOS-6.3-i386-bin-DVD1.iso"
    loopback loop (hd0,1)/$isofile
    linux (loop)/isolinux/vmlinuz method=hd:/dev/sdb1:/centos
    initrd (loop)/isolinux/initrd.img 
}
```
***
##3.目录结构

***
####CentOS
```
/centos/CentOS-6.3-i386-bin-DVD1.iso
/centos/images/好多个内容
**images目录是从CentOS的镜像中复制出来的**
```
####Fedora或者Ubuntu
```
/Ubuntu.iso
/Fedora-17-i386-DVD.iso
```
