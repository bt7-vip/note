# fedora 40安装Remmina

在linux桌面端，安装远程桌面工具远程windows主机，在安装时遇到一些有意思的事情

## 1：源

在Remmina网站，有各个发行版的安装命令，在fedora/redhat是这样安装

```shell
dnf copr enable hubbitus/remmina-next
dnf upgrade --refresh 'remmina*' 'freerdp*'
```

实际执行的时候，第一条命令就失败。

## 2：手动添加源

在wiki中有redahat/centos的安装方法，是使用epel源安装。

```
wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -ivh epel-release-latest-7.noarch.rpm
```

> A more updated version is available thanks to @castorsky through a [COPR repo](https://copr.fedorainfracloud.org/coprs/castor/remmina/)

我看到CORE repo，正好之前在安装scrcpy时，添加过core源，所以就点进去看了一下

页面提供的copr源

```
[copr:copr.fedorainfracloud.org:castor:remmina]
name=Copr repo for remmina owned by castor
baseurl=https://download.copr.fedorainfracloud.org/results/castor/remmina/epel-8-$basearch/
type=rpm-md
skip_if_unavailable=True
gpgcheck=1
gpgkey=https://download.copr.fedorainfracloud.org/results/castor/remmina/pubkey.gpg
repo_gpgcheck=0
enabled=1
enabled_metadata=1
```

可以看到是提供的epel 8版本，在页面`https://download.copr.fedorainfracloud.org/results/castor/remmina`中，也只看到`epel-8-x86_64`,虽然源指向的发行版是8，但也许有用呢。

## 3：安装

使用`sudo dnf install remmina`,发现返回的安装包只有几M，感觉不对，于是使用

```
sudo dnf install 'remmina*'
```

安装有几百M，并且有插件，看包的版本，发现对应fc40和el8的包都有。确认安装。

## 4：安装成功

desktop是fedora40-xfce，安装后远程windows正常使用。

## 5：共享文件夹

对于无法通过复制粘贴来传送文件的问题，只能通过共享文件夹来解决

在配置页面，有个共享文件夹，编写格式为：

```shell
name,path;
# 示例
sharedir,/home/usr/share;
# 其中sharedir会在对端设备显示出来，
# /home/usr/share为本地目录
```

  之前安装软件的时候就遇到fedora无法安装epel源，不知道现在fedoar是怎么处理epel源的，明明epel是fedora在维护，结果自己的发行版无法简单顺利的使用源，有点魔幻。