# VirtualBox 的使用
***
## 1.安装
***

* 如果无法启动虚拟机，需要重新编译vbox的内核模块
* 如果安装的时候安装了开发环境，那么kernel-devel和kernel-headers不需要安装了

```
/etc/init.d/vboxdrv setup
```
* 如果没有安装，下面安装的kernel-devel和当前内核不一定匹配
* 最好升级一下内核，然后从最新得内核中启动，再安装

```
yum install kernel kernel-devel kernel-headers
从最新内核重启
/etc/init.d/vboxdrv setup
```

***
## 2.文件共享
***
#### Windows 7虚拟机
* 开启虚拟机后，选择菜单栏Devices->Insert Guest Addtion CD Image
* 然后在Windows中安装这个软件。
* 关机后，选择虚拟机，右键Settings-> Shared Folders
* 点击右边的带加号的文件夹

  * 选择路径，比如说叫做"windows_share"，勾选Auto-mount,确定

* 打开计算机，点左边的计算机(就是显示了C盘，D盘的时候)
* 在菜单栏上，点击"映射网络驱动器"
* 选择盘符，和文件夹右边的"浏览"->VBOXSVR->\\VBOXSVR\windows_share，确定
* 这样就可以想C盘之类得一样使用这个网络驱动器了。

***
## 3.虚拟机瘦身
***
#### Windows虚拟机

* 1.在虚拟机中

下载Sysinternals Suite并执行

```
sdelete –z
```

* 2.在CentOS上

```
VBoxManage modifyhd mydisk.vdi --compact
```

####Linux虚拟机

* 1.在虚拟机中

```
sudo dd if=/dev/zero of=/EMPTY bs=1M
sudo rm -f /EMPTY
```

* 2.在CentOS上

```
VBoxManage modifyhd mydisk.vdi --compact
```

***
## 4.Cannot register the hard disk，because a hard already exists
***

* 重新指定一个UUID就可以了

```
VBoxManage  internalcommands sethduuid mydisk.vdi
```
