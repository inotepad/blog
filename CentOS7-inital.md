
#  CentOS 7 日常使用配置
***
## 安装提示
***

* 选择Gnome Desktop后，把开发环境也勾选上，这样可以方便编译一些软件。
* 安装完成之后可能没有windows的引导
 * 将以下代码添加进/etc/grub.d/40_custom

```
menuentry "Windows"{
    set root=(hd0,1)
    chainloader +1
}
```
* 然后更新引导

```sh
grub2-mkconfig -o /boot/grub2/grub.cfg
```

***
## 日常使用
***
#### 1. 中文输入法

* System Tools->**Settings**->**region and language**->点击下面的**+**->选择Chinese->pinyin 和下面的Add按钮
* 使用Window键+空格键来切换中文和英文输入法
* 如果处于中文输入法下，可以使用shift键来切换中英文输入

#### 2. 第三方源

##### epel

```sh
rpm -Uvh http://download.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm
```

* 如果上面的命令不能使用，去下面的网页看最新的命令

```sh
http://fedoraproject.org/wiki/EPEL/FAQ#howtouse
```

##### nux-desktop

```sh
rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-1.el7.nux.noarch.rpm
```
#### 3. ntfs
```sh
需要epel
yum install ntfs-3g
```
#### 4. 自动挂载硬盘

* 通过blkid可以看到每个分区的UUID

```sh
# blkid
/dev/sda6: UUID="EE909F16909EE47D" TYPE="ntfs" 
/dev/sda7: UUID="8db1afdd-53eb-4a87-b975-17d599fe5e8c" TYPE="ext4"
```
##### 挂载ext4等分区
* 如果要将/dev/sda7作为数据盘，希望能够自动挂载 

```sh
# mkdir /mnt/data
$ ln -s /mnt/data ~/data
```

* 将下面内容添加到/etc/fstab
* **注意修改UUID和路径**

```
UUID=8db1afdd-53eb-4a87-b975-17d599fe5e8c /mnt/data               ext4    defaults        0 0
```

##### 挂载ntfs等分区

* 如果要挂载ntfs的分区也是类似的
* 以/dev/sda6为例

```sh
# mkdir /mnt/WinD
$ ln -s /mnt/WinD ~/WinD
```

* 将下面内容添加到/etc/fstab
* **注意修改UUID和路径**

```
UUUID="EE909F16909EE47D  /mnt/WinD          ntfs-3g    defaults        0 0
```

* **注意**
* 如果是使用**Windows8+CentOS双系统**的话，请注意Windows8不要使用关机，要使用重启
* Windows8关机并不会完全关机，这时CentOS自动挂载ntfs的分区，就会出现无法启动的异常
* **这种情况下出现的问题**
* 如果ntfs的盘无法手动挂载，进入Windows8,**重启**（不是关机）
* 如果设置了ntfs的盘自动挂载后，CentOS无法正常启动，进入Windows8，**重启**(不要关机)

#### 5. 中文支持

##### shell中文支持

**terminal**->**set character encoding**

##### vim中文支持

修改.vimrc

````
set encoding=utf-8
set termencoding=utf-8
set formatoptions+=mM
set fencs=utf-8,gbk
````

##### gedit3中文支持

```
gsettings set org.gnome.gedit.preferences.encodings auto-detected "['UTF-8','CURRENT','GB18030','ISO-8859-15','UTF-16']"
```

#### 6. chrome

官网下载

#### 7. 音乐播放器audacious

```sh
需要nux-desktop
yum install audacious audacious-plugins-freeworld
```

#### 8. 视频播放器VLC

##### 安装 VLC

```sh
yum install vlc
yum remove totem
```

##### 如果VLC字幕乱码
*  首先启动VLC，按Ctrl+P,左下角的显示设置 选 全部
*  依次点开：视频－字幕／OSD－文本渲染器 右侧的字体栏中，选择一个中文字体。（micro)
*  接着点开：输入／编码－其它编码器－字幕 右侧的 字幕文本编码 选 GB18030
*  然后 把 自动检测 UTF－8  和字幕 格式化字幕 前面的勾去掉

#### 9. WPS的安装
* 从WPS的官网下载**alpha版本**安装包wps-office-9.1.0.4945-1.a16p3.i686.rpm
* 使用yum install,不要使用rpm -ivh

```sh
yum install wps-office-9.1.0.4945-1.a16p3.i686.rpm
```

* 安装之后还有公式的字体问题下载symbol－fonts_all.deb
* 解压之后可以找到字体，将这些字体复制到~/.fonts中(如果没有该文件夹，就新建一个)

```sh
cd ~/.fonts
mkfontscale
mkfontdir
fc-cache
```

#### 10. 画图软件gimp

```sh
yum install gimp
```
#### 11. BT软件

```sh
yum install transmission
```
#### 12. 共享文件
* 如果需要共享~/Documents中的文件

```sh
cd ~/Documents
python -m SimpleHTTPServer 8080
firewall-cmd --permanent --zone=public --add-port=8080/tcp
```
* 别人在浏览器中访问这个server就可以获取目录下的文件

#### 13. 使用NoMachine进行远程桌面连接
* NoMachine既可以在Windows上使用，也可以在Linux上使用

```sh
需要epel和nux-desktop
yum -y install xrdp tigervnc-server
systemctl start xrdp.service
netstat -antup | grep xrdp
systemctl enable xrdp.service
firewall-cmd --permanent --zone=public --add-port=3389/tcp
firewall-cmd --reload
```

***
## 开发环境
***
#### 1. 开发的依赖
```sh
yum groupinstall 'Development Tools'
```

#### 2. vim 插件
neocomplcache和tagslist
#### 3. Java开发环境配置

```sh
yum install java-1.7.0-openjdk
yum install java-1.7.0-openjdk-devel
echo 'JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk' >> ~/.bashrc
```
#### 4. Java IDE配置
##### Eclipse

* Eclipse的快捷方式
 * 图片icon.xmp太大,可以右键resize(33%),然后保存为icon.png
* 将下面的内容写入文本，命名为Eclipse.desktop
 * 其中的Exec是启动的eclipse的路径
 * Icon是图标所在的路径

```
[Desktop Entry]
Version=1.0
Name=Eclipse
Exec=/home/young/software/eclipse/eclipse
Terminal=false
Icon=/home/young/software/eclipse/icon.png
Type=Application
Categories=Development;
```

* 编辑器的插件(高亮插件)
 * http://blog.csdn.net/ichsonx/article/details/9148497

##### Idea
* Idea.Desktop
 * 其中的Exec是启动的eclipse的路径
 * Icon是图标所在的路径

```
[Desktop Entry]
Version=1.0
Name=Idea
Exec=/home/young/software/idea/idea-IC-135.1230/bin/idea.sh
Terminal=false
Icon=/home/young/software/idea/idea-IC-135.1230/bin/idea.png
Type=Application
Categories=Development;
```
#### 5. 安装R和RStudio
* R

```sh
yum install R
```
* R Studio

```
http://www.rstudio.com/products/rstudio/download/
```

#### 6. 安装Racket
* 编译查看src/READEME
* 安装好，快捷方式share/appliction，修改图标和程序路径(变成绝对路径)
