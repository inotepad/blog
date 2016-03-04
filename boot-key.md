# 其他制作启动盘的方法
***
##1.Fedora LiveCD
***
* ####使用软件liveusb-creator
https://fedorahosted.org/liveusb-creator/
* ####dd
```
sudo dd if=xxx.iso of=/dev/sdb bs=1M
```

***
##2.Fedora DVD,CentOS DVD
***
####软件UNetbootin
*unetbootin.sourceforge.net*

* 1.将CentOS.DVD.iso中除Packages外的其他文件做成一个新的ISO
* 2.用UNetbootin把新的ISO写入U盘
* 3.把CentOS.DVD.iso一个文件复制到U盘下
解释：CentOS安装时要求整个完整的ISO，是为了里面的软件包。

其实，第一步可以跳过不做的，直接用第二步，不过要这样第二步的时间会很长，而且如果是8G，4G的U盘还要把Packages文件夹手动删除，也是很耗时的。安装时CentOS是在ISO文件中找软件包的，这个Packages文件夹纯粹是浪费空间

附：

```
创建目录centos

mkisofs -r -o XXX.iso  -graft-point /=centos

单个目录制作ISO
mkisofs -r -o XXX.iso /home

多个目录制作ISO
mkisofs -r -o XXX.iso  -graft-point images=images isolinux=isolinux

mkisofs -r -o XXX.iso  -graft-point /home=/home /root=/root
注意-graft-point后面的参数是isodir=systemdir。
```

***
##3.Ubuntu
***
http://www.ubuntu.org.cn/download/help/create-a-usb-stick-on-ubuntu
http://www.ubuntu.com/download/help/create-a-usb-stick-on-windows

***
##4.Debian(Linux)
***
http://www.debian.org/releases/stable/i386/ch04s03.html.en
```
cat debian.iso >/dev/sdb之类的
```
