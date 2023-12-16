# 管理vm虚拟机

新创建的vm虚拟机，无法在pve gui中方便查看、管理vm，比如不知道虚拟机的IP，关机/重启失败等情况。这种情况下，需要在vm中的系统中安装与宿主机通信的qemu-guest，以便宿主机可以正常管理vm。

### linux虚拟机：

```shell
yum install qemu-guset-agent -y
systemctl start qemu-guset-agent
systemctl enable qemu-guest-agent
```

### windosw虚拟机：

挂载windows驱动安装

```powershell
https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/virtio-win.iso
添加光驱》windows运行 virtio-win-guest-tools.exe》一路下一步
```



### 在vm的管理界面：

在参数处将Qume Guest开启，关机》开机，之后就可以通过Cloud-init配置



### vm强制重启

在未安装qemu-guset-agent的情况下，重启关机失败时，按以下步骤

```shell
rm -rf /run/lock/qemu-server/lock-ID.conf
qm unlock ID
qm stop ID
```



