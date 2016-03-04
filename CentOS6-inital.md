# CentOS 6 日常使用配置
***
##日常使用
***
###1.中文输入法

```
im-chooser
```
然后use ibus,选择chinese 中得pinyin ，add就可以了

###2.第三方源
#####epel

http://fedoraproject.org/wiki/EPEL/FAQ#howtouse

```
rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
```

###3.ntfs
```
yum install ntfs-3g
```
###4.chrome
* 自动安装脚本

```
 http://chrome.richardlloyd.org.uk/install_chrome.sh
```
* 然后使用gedit编辑install—chrome.sh

```
用find功能查找并将其中的
http://omahaproxy.appspot.com
改为
https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm。
```
* 打开终端，依次执行

```
chmod +x install_chrome.sh
./install_chrome.sh
```

###5.中文支持

#####shell中文支持

**terminal**->**set character encoding**

#####vim中文支持

修改.vimrc

````
set encoding=utf-8
set termencoding=utf-8
set formatoptions+=mM
set fencs=utf-8,gbk
````

#####gedit2中文支持

```
gconftool-2 --set --type=list --list-type=string /apps/gedit-2/preferences/encodings/auto_detected "[UTF-8,CURRENT,GB18030,ISO-8859-15,UTF-16]"
```

###6.影音播放

```
yum install gstreamer-plugins-bad gstreamer-ffmpeg gstreamer-plugins-ugly -y
```

####7.BT软件

```
yum install transmission
```

***
##开发环境
***
###1.开发的依赖
```
yum groupinstall 'Development Tools'
```
###2.vim 插件
neocomplcache和tagslist
###3.eclipse
#####编辑器的插件(高亮插件)
http://blog.csdn.net/ichsonx/article/details/9148497
