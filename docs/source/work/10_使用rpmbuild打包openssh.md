# 2024/5/10 使用rpmbuild 打包 openssh

openssh源码自带spec文件，我们可以直接使用自带spec文件了解和学习打包流程。

## 环境准备

### 1：准备虚拟机

安装好系统后ssh登录。

```powershell
宿主系统：Proxmox
虚拟机环境：centos-cloud-7
```

> 自从学会了在pve中挂载cloud镜像创建虚拟机，结合快照，用的是越来越爽了。速度是真的快，而且cloud镜像资源占用少，比为了准备一个原始的环境重新安装一个系统简直是质的飞跃(*^_^*)。

### 2：软件安装

```powershell
sudo yum install rpm-build rpmdevtools rpmlint wget zlib-devel openssl-devel gcc perl-devel pam-devel unzip libXt-devel imake gtk2-devel perl -y
```

其他环境可能会遇到需要开启PowerTools源的情况。

## 创建文件夹

```powershell
cd ~
rpmdev-setuptree
cd rpmbuild/SOURCES
#下载源码包
wget https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-9.3p2.tar.gz
#解压并复制spec文件
tar -xvf openssh-9.3p2.tar.gz
cp openssh-9.3p2/contrib/redhat/openssh.spec ../SPECS
cd ../SPECS
```

## 开始制作包

先跑，出错了在改

```powershell
rpmbuild -ba openssh.spec
```

### 问题1：补充ssh-askpass文件

```powershell
error: File /home/test/rpmbuild/SOURCES/x11-ssh-askpass-1.2.4.1.tar.gz: No such file or directory
```


```powershell
wget -P ~/rpmbuild/SOURCES/ https://src.fedoraproject.org/repo/pkgs/openssh/x11-ssh-askpass-1.2.4.1.tar.gz/8f2e41f3f7eaa8543a2440454637f3c3/x11-ssh-askpass-1.2.4.1.tar.gz
#不用担心版本，2004年的包，到现在都没更新过。
```

### 问题2：openssl版本报错

```powershell
openssl-devel < 1.1 is needed by openssh-9.3p1-1.el7.BCLinux.x86_64
```

查看spec文件

```powershell
BuildRequires: openssl-devel >= 1.0.1
BuildRequires: openssl-devel < 1.1
```

小于1.1应该是针对1.1及其小版本，为了区别3.0版本，所以可以修改为

```powershell
BuildRequires: openssl-devel < 1.1.1
```

修改后再BC8.2的环境，是可以继续打包。

但是再centos7中，还是会出错，其他位置也并未指定openssl版本号，所以 就很奇怪。能继续执行的有两种方法

```powershell
#1
BuildRequires: openssl-devel > 1.1
#2注释此项
#BuildRequires: openssl-devel < 1.1
```

再次开始编译

不出意外的话，rpm包已经好了。

```powershell
$ tree RPMS/
RPMS/
└── x86_64
    ├── openssh-9.3p2-1.el7.x86_64.rpm
    ├── openssh-askpass-9.3p2-1.el7.x86_64.rpm
    ├── openssh-askpass-gnome-9.3p2-1.el7.x86_64.rpm
    ├── openssh-clients-9.3p2-1.el7.x86_64.rpm
    ├── openssh-debuginfo-9.3p2-1.el7.x86_64.rpm
    └── openssh-server-9.3p2-1.el7.x86_64.rpm

1 directory, 6 files
```

## 安装测试

包在打包好后，需要安装测试，确认是否出现其他问题

跨版本升级openssh，会出现配置文件被替换，已存在的key等文件因为不同版本对权限的定义不同，导致安装后sshd启动失败，我们根据失败的原因，重新修改spec文件并对安装包执行的动作进行修改。

我们先在新系统中进行安装测试

```powershell
#使用lrzsz或者其他方式，将包传送到系统中
#开始安装
sudo yum localupdate openssh-*.rpm -y
#查看ssh版本号
ssh -V
#查看ssh服务是否启动
sudo systemctl status sshd
```

### 问题1：

```powershell
Unable to load host key "/etc/ssh/ssh_host_ed25519_key": bad permissions
```

我们先看下`/etc/ssh/ssh_host_ed25519_key`的权限

```powershell
$ sudo ls -la /etc/ssh/ssh_host_ed25519_key
-rw-r-----. 1 root ssh_keys 387 May  9 06:59 /etc/ssh/ssh_host_ed25519_key
```

没有提示0640权限是大还是小，我们按越小越安全的原则，先将文件改成0600测试一下

```powershell
$ sudo chmod 0600 /etc/ssh/ssh_host_ed25519_key
$ sudo systemctl restart sshd
$ sudo systemctl status sshd
```

这次是成功启动，但是status回显还是有其他文件显示权限问题

```powershell
Unable to load host key "/etc/ssh/ssh_host_ecdsa_key": bad permissions
Unable to load host key: /etc/ssh/ssh_host_ecdsa_key
[  OK  ]
```

那我们就把/etc/ssh/下的所有key权限缩小

```powershell
$ sudo chmod 0600 /etc/ssh/*key
$ sudo systemctl restart sshd
$ sudo systemctl status sshd
```

没有出现明显的告警，有一个公钥未找到的提示。

### 问题2：

登录测试，在没有其他告警后，新开窗口进行登录测试

登录失败，此时在旧窗口查看sshd的信息

```powershell
$ sudo systemctl status sshd
```

会发现一条记录

```powershell
error: Could not get shadow information for xxx
```

修改配置文件

```powershell
$vim /etc/ssh/sshd_config
...
usePAM yes
...
```

### 问题3：

再次重启ssh服务并重新测试登录，还是失败，查看ssh日志

```powershell
PAM unable to dlopen(/usr/lib64/security/pam_stack.so): /usr/lib64/security/pam_stack.so: cannot open shared object f...r directory
PAM adding faulty module: /usr/lib64/security/pam_stack.so
fatal: Access denied for user xxx by PAM account configuration [preauth]
```

pam模块出现问题，我们升级的ssh，怎么和pam扯上关系？？

看下提示的库文件，还不存在，看下/etc/pam.d/的文件信息，可以看到sshd的日期被更改，对比新旧配置，会发现很多配置配指向了`pam_stack.so`这个库，系统中并不存在这个库，并且，这个库在仓库中找不到提供的包。

经确认，这个pam认证方式已经被淘汰，现在使用更新的认证方式，就是原配置中的认证，那么我们只需要将旧配置还原就行。

## 优化打包

在安装测试时，我们发现了3个问题，一个是权限问题，另外两个是配置问题。在spec中，我们可以对文件进行操作，这样，我们在执行安装包时，就可以将文件先备份，再升级软件，再将配置还原等自动化操作。

spec提供了一些语法，可以在安装前及安装后进行一些操作，我们将spec文件进行修改

```powershell
$ vi openssh.spec
...
%if ! %{no_x11_askpass}
%setup -q -a 1
%else
%setup -q
%endif

#升级前备份旧配置文件
%pre
if [ -f %_sysconfdir/ssh/sshd_config ]; then
    cp %_sysconfdir/ssh/sshd_config %_sysconfdir/ssh/sshd_config.oldfile
fi

if [ -f %_sysconfdir/pam.d/sshd ]; then
    cp %_sysconfdir/pam.d/sshd %_sysconfdir/pam.d/sshd.oldfile
fi
#升级后还原配置文件
%post
mv %_sysconfdir/pam.d/sshd.oldfile %_sysconfdir/pam.d/sshd
mv %_sysconfdir/ssh/sshd_config.oldfile %_sysconfdir/ssh/sshd_config
chmod 0600 %sysconfdir/ssh/*key
...
```

再次进行打包和安装测试，没有做任何操作，可以发现新开窗口已经可以直接正常登录，并且文件权限和配置也是符合期望。

## 流程整理

创建rpm文件夹

安装编译相关依赖

将源码下载到SOURCES目录

编写spec文件

打包测试

解决打包问题

安装测试

解决安装问题

优化打包

再次测试软件功能

这次流程仅是针对openssh的简单测试，还有其他功能例如askpass这些没有进行安装和测试，实际使用中，还需要更全面的功能测试，例如sftp功能，ssh-id等功能。并且再需要时多次重复以上步骤来解决不同的问题。
