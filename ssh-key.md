#SSH Key的使用和管理
***
### 0.概述
***
* 使用

  登录git

  无需密码SSH登录登录服务器

* 管理

  生成密钥文件的时候，注意密钥文件命名

  使用~/.ssh/config来管理不同网站使用不同的密钥文件

***
### 1.git 配置SSH Key
***

* 生成密钥文件时修改文件命名，方便管理之后的管理

  ~/.ssh/id_rsa_github

```
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/young/.ssh/id_rsa): /home/young/.ssh/id_rsa_github
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/young/.ssh/id_rsa_github.
Your public key has been saved in /home/young/.ssh/id_rsa_github.pub.
```

***
### 2.自动登陆服务器配置
***

* 生成密钥文件时修改文件命名，方便管理之后的管理

  ~/.ssh/id_rsa_server

```
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/young/.ssh/id_rsa): /home/young/.ssh/id_rsa_server 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/young/.ssh/id_rsa_server.
Your public key has been saved in /home/young/.ssh/id_rsa_server.pub.

$ ssh-copy-id -i ~/.ssh/id_rsa_server.pub username@server
```
***
### 3.多个ssh-key的管理
***
* 编辑~/.ssh/config

```
# github
Host github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_github
# xxx的gitlab 
Host gitlab.xxx.com
    HostName xxx.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_xxx
# server 
Host server 
    HostName server
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_server
Host *
    PreferredAuthentications password
```
