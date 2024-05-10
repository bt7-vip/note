# 2024/4/30: fedora跨版升级系统故障清除记录

  备用linux系统一直使用的是fedora，安装的版本是38。之前改造工作机，原本想安装新的虚拟机，鉴于环境迁移麻烦，索性直接将虚拟机从vm迁移的pve继续运行，省心。上午看os-release文件时看到支持周期到5月14号结束，于是尝试升级到fedora 40。使用的是XFCE桌面版，并不是一键升级，期间遇到一些问题，一步一步解决。

### 一：升级当前系统

```shell
sudo dnf upgrade --refresh
sudo reboot
```

在这里出现了问题，我分不清`update`和`upgrade`，所以我先执行的`sudo dnf update `然后执行`sudo dnf upgrade -y`,执行`update`时有一个报错：

>  问题: 无法为软件包安装最佳更新候选 xfce4-whiskermenu-plugin-2.7.3-1.fc38.x86_64
>
>   - nothing provides libaccountsservice.so.0()(64bit) needed by xfce4-whiskermenu-plugin-2.8.3-1.fc38.x86_64 from updates

但是执行是成功的，没有在意继续执行了`reboot`,开机后执行`sudo dnf upgrade`,依旧报错，我搜索了一下这个包`libaccountsservice.so.0()(64bit)`,找到这个[连接]([accountsservice-libs-23.11.69-2.fc38.x86_64.rpm Fedora 38 Download (pkgs.org)](https://fedora.pkgs.org/38/fedora-x86_64/accountsservice-libs-23.11.69-2.fc38.x86_64.rpm.html)) ，下载包并进行安装

```shell
sudo dnf localinstall accountsservice-libs-23.11.69-2.fc38.x86_64.rpm
```

很奇怪为什么仓库没有这个包。

然后在重新执行`sudo dnf upgrade --refresh`顺利升级`xfce4-whiskermenu-plugin`

### 二：安装dnf插件

```shell
sudo dnf install dnf-plugin-system-upgrade
```

### 三：下载升级包

```shell
sudo dnf system-upgrade download --releasever=40
```

执行后检测报错：

> 错误：
>  问题 1: package rpmfusion-free-release-38-1.noarch from @System requires system-release(38), but none of the providers can be installed
>   - fedora-release-xfce-38-36.noarch from @System  does not belong to a distupgrade repository
>   - 安装的软件包的问题 rpmfusion-free-release-38-1.noarch
>  问题 2: package fedora-release-xfce-38-36.noarch from @System requires fedora-release-common = 38-36, but none of the providers can be installed
>   - package rpmfusion-nonfree-release-38-1.noarch from @System requires system-release(38), but none of the providers can be installed
>   - fedora-release-common-38-36.noarch from @System  does not belong to a distupgrade repository
>   - 安装的软件包的问题 rpmfusion-nonfree-release-38-1.noarch
> (尝试添加 '--skip-broken' 来跳过无法安装的软件包)

是包不兼容，那先将包删掉

```shell
#确认包
sudo rpm -qa | grep "rpmfusion-free-release-38-1.noarch"
#卸载
sudo dnf remove rpmfusion-free-release-38-1
#确认包
sudo rpm -qa | grep "edora-release-xfce-38-36.noarch"
#卸载
sudo dnf remove fedora-release-xfce-38-36
```

在开始升级

```shell
sudo dnf system-upgrade download --releasever=40
```

顺利开始跑码。

### 四：开始升级

```shell
dnf system-upgrade reboot
```

升级很慢，需要耐心等待

![image-20240429165210289](./image-20240429165210289.png)
