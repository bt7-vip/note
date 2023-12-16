shell配置代理
=================
dnf install proxychains-ng

vim /etc/proxychains.conf

```
socks5	IP port
```

需要代理的命令前加proxychains4

如：

```shell
sudo proxychains4  dnf update -y
proxychains4 sudo dnf update -y
```



不知道为什么，sudo放前和放后的问题
