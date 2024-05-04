# 2024/4/29: wireguard流量转发配置

在搭建过多个wireguard网络后，发现对iptables的理解还是很浅显，这里记录一下2个转发流量的iptables语句。便于参考

```shell
PostUp = iptables -I FORWARD -i wg0 -j ACCEPT; iptables -I FORWARD -o wg0 -j ACCEPT; iptables -I INPUT -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -D INPUT -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

```
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

配置参考

windows

```shell
[Interface]
PrivateKey = 私钥
ListenPort = 端口
Address = 192.168.50.1/32
DNS = 192.168.1.1

[Peer]
PublicKey = 公钥
AllowedIPs = 192.168.50.10/32
PersistentKeepalive = 25

[Peer]
PublicKey = 公钥
AllowedIPs = 192.168.13.0/24
Endpoint = baidu.com:63280
PersistentKeepalive = 25
#两种连接方式，一个是本段是公网可以打开端口，peer1在net后可以连进来
#对端是公网有端口，本段是否有公网无关，没有的情况下ListenPort = 端口不要添加。连peer2.
```

linux

```shell
[Interface]
PrivateKey = 私钥
ListenPort = 端口
Address = 192.168.50.1/32
DNS = 192.168.1.1
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
[Peer]
PublicKey = 公钥
AllowedIPs = 192.168.50.10/32
PersistentKeepalive = 25
```



## FAQ

对于局域网转发，有个坑

比如： A客户端=====> B服务端{局域网}

此时想要在A访问B局域网，因为B已经开启转发，所以B的配置中peer用户，不需要将局域网添加进去，会造成回环。导致局域网流量无法到达局域网关。

在A的peer中，将向要访问的局域网端添加到peerB中，由A将访问的流量转发到B中。
