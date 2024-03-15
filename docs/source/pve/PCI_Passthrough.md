# Proxmox VE PCI直通

## 一修改配置文件

1：编辑grub文件

```
nano /etc/default/grub
#GRUB_CMDLINE_LINUX_DEFAULT="quiet"
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt video=efifb:off"
```

`video`参数在proxmox文档中，并未显示需要添加，在其他文章中发现`video=vesafb:off video=efifb:off`,不确定这个参数和是否能看到login有关

2：添加module

```
nano /etc/modules
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd #6.2版本以上的kernelsvfio模块已经包含vfio_virqfd，无需单独添加
```

3：update initramfs

```
update-initramfs -u -k all
```

> root@pve:~# update-initramfs -u -k all
> update-initramfs: Generating /boot/initrd.img-6.5.11-4-pve
> Running hook script 'zz-proxmox-boot'..
> Re-executing '/etc/kernel/postinst.d/zz-proxmox-boot' in new private mount namespace..
> No /etc/kernel/proxmox-boot-uuids found, skipping ESP sync.

注意这个回显是正常的，skipping不是问题。

4：禁用显卡驱动

```
echo "blacklist amdgpu" >> /etc/modprobe.d/blacklist.conf
echo "blacklist radeon" >> /etc/modprobe.d/blacklist.conf
```

5：重启系统

```
reboot
```

## 二确认是否生效

使用以下命令查询

```
lspci -nnk
```

可以看到显卡的信息`Kernel driver in use: vfio-pci`

> 02:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Polaris 20 XL [Radeon RX 580 2048SP] [1002:6fdf] (rev ef)
>         Subsystem: XFX Pine Group Inc. Polaris 20 XL [Radeon RX 580 2048SP] [1682:e580]
>         Kernel driver in use: vfio-pci
>         Kernel modules: amdgpu