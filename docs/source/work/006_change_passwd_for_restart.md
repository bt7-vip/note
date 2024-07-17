# 在引导时重置root密码

time: 2024/4/27

流程

### 重启系统，在 GRUB 2 引导屏幕上按 **e** 键中断引导过程

此时会出现内核引导参数

```shell
load_video
set gfx_payload=keep
insmod gzio
linux ($root)/vmlinuz-4.18.0-80.e18.x86_64 root=/dev/mapper/rhel-root ro crash\
kernel=auto resume=/dev/mapper/rhel-swap rd.lvm.lv/swap rhgb quiet
initrd ($root)/initramfs-4.18.0-80.e18.x86_64.img $tuned_initrd
```

## 方法一

### 进入以 **linux** 开头的行的末尾。

```shell
linux ($root)/vmlinuz-4.18.0-80.e18.x86_64 root=/dev/mapper/rhel-root ro crash\
kernel=auto resume=/dev/mapper/rhel-swap rd.lvm.lv/swap rhgb quiet
```

按 **Ctrl+e** 键跳到这一行的末尾。

### 在以 `linux` 开头的行的最后添加 `rd.break`。

```shell
linux ($root)/vmlinuz-4.18.0-80.e18.x86_64 root=/dev/mapper/rhel-root ro crash\
kernel=auto resume=/dev/mapper/rhel-swap rd.lvm.lv/swap rhgb quiet rd.break
```

### 按 **Ctrl+x** 使用更改的参数启动系统。

此时会出现 `switch_root` 提示符。

### 将文件系统重新挂载为可写：

```shell
mount -o remount,rw /sysroot
```

文件系统以读写模式挂载到 `/sysroot` 目录中。将文件系统重新挂载为可写才可以更改密码。

### 进入 `chroot` 环境

```shell
chroot /sysroot
```

此时会出现 `sh-4.4#` 提示符。

### 重置 `root` 密码

```shell
passwd
```

按照命令行中的步骤完成 `root` 密码的更改。

需要修改其他用户就添加用户名

```shell
passwd USER
```

除了修改密码，在chroot环境下可以进行其他操作，比如修改`/etc/fstab`，对挂载恢复。

### 在下次系统引导时启用 SELinux 重新标记进程

```shell
touch /.autorelabel
```

### 退出 `chroot` 环境：

```shell
exit
```

### 退出 `switch_root` 提示符：

```shell
exit
```

等待 SELinux 重新标记过程完成。请注意，重新标记一个大磁盘可能需要很长时间。系统会在这个过程完成后自动重启。

## 方法二

方法一是针对centos5，6比较老的版本，新版本可以在grub引导时直接修改，进入`chroot`

将`ro`修改为`rw`,在末尾添加`init=/bin/bash`

```shell
linux ($root)/vmlinuz-4.18.0-80.e18.x86_64 root=/dev/mapper/rhel-root rw crash\
kernel=auto resume=/dev/mapper/rhel-swap rd.lvm.lv/swap rhgb quiet init=/bin/bash
```

不同发行版这一行会有差异，饭这两处修改相同。修改后按下`Ctrl+x`，几秒后就会进入`#sh-4`,此时就可以进行其他操作了。

## F&A

如果修改`grub`后，还是进入救援模式，输入任何命令都是提示未找到。可以`ls`看下,这个命令一般都是可用状态，如果出现`sysroot`，那么，按照方法一的步骤，重新挂载`sysroot`和`chroot`过去，标识符可能不会变，还是`#`,不过`ls`因该时可以看到`/`下的文件了。
