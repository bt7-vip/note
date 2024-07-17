#  build openssh rpm包

time: 2024/1/6

有个rpm升级openssh，也得有打包方法，感谢openssh自带sepc文件。

## 创建文件夹

```
mkdir -p ./rpmbuild/{SOURCE,SPECS}
cd rpmbuild/SOURCE
#下载源码包
wget https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-9.3p2.tar.gz
#解压并复制spec文件
tar -xvf openssh-9.3p2.tar.gz
cp openssh-9.3p2/contrib/redhat/openssh.spec .
#安装软件
yum install rpm-build zlib-devel openssl-devel gcc perl-devel pam-devel unzip libXt-devel imake gtk2-devel -y
```

不出意外的话应该安装失败，提示imake没有源包，需要开启PowerTools源。

```
yum repolist
```

## 开始制作包

```
rpmbuild -ba openssh.spec
```

先跑，出错了在改

### 1：补充ssh-askpass文件

```
error: File /root/rpmbuild/SOURCES/x11-ssh-askpass-1.2.4.1.tar.gz: No such file or directory
```


```
wget https://src.fedoraproject.org/repo/pkgs/openssh/x11-ssh-askpass-1.2.4.1.tar.gz/8f2e41f3f7eaa8543a2440454637f3c3/x11-ssh-askpass-1.2.4.1.tar.gz
#不用担心版本，2004年的包，到现在都没更新过。
```

### 2：openssl版本报错

```
openssl-devel < 1.1 is needed by openssh-9.3p1-1.el7.BCLinux.x86_64
```

查看spec文件

```
BuildRequires: openssl-devel >= 1.0.1
BuildRequires: openssl-devel < 1.1
```

小于1.1应该是针对1.1及其小版本，为了区别3.0版本，所以可以修改为

```
BuildRequires: openssl-devel < 1.1.1
```

开始编译

不出意外的话，rpm包已经好了。
