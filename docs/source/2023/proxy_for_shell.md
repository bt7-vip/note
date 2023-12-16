# 为shell设置代理

安装软件
dnf install proxychains-ng
编辑配置文件
vim /etc/proxychains.conf

```
socks5	IP port
```
使用方法
需要代理的命令前加proxychains4

如：

```shell
sudo proxychains4  dnf update -y
proxychains4 sudo dnf update -y
```



不知道为什么，sudo放前和放后的问题
