# 在pve中，通过SDN快速创建局域网

## 场景

在只有一个公网IP并分配给PVE宿主机的情况下，创建的虚拟机需要通过创建虚拟局域网获取内网ip，可以使用pve8以后新添加的SDN功能，快速创建虚拟机的局域网，并配置dhcp

## 安装依赖

```shell
# 更新系统
apt update && apt upgrade -y 
# 安装所需软件
apt install dnsmasq frr-pythontools
# 配置dnsmasq开机启动
systemctl disable --now dnsmasq
```



## 创建zones

在DataCenter》SDN》Zones创建一个simple网络，选中automatic DHCP自动分配网址

![image-20240204161048777](./typora-user-images/image-20240204161048777.png)

## 创建VNet

在VNets创建一个Vnet，zone选择上面创建的simple

![image-20240204161213828](./typora-user-images/image-20240204161213828.png)

## 创建subnets

选中创建好的vnet，s在subnets创建局域网的网段和dhcp范围，勾选SNAT使局域网可以连互联网。

![image-20240204161310022](./typora-user-images/image-20240204161310022.png)

![image-20240204161324300](./typora-user-images/image-20240204161324300.png)

DHCP一定要设置，分配。

## 应用SDN

创建好后回到SDN，点击apply应用。

![image-20240204161628952](./typora-user-images/image-20240204161628952.png)

等待几秒，status变为available，就可以在虚拟机添加网络使用了

![image-20240204161905567](./typora-user-images/image-20240204161905567.png)

