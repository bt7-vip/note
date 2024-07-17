# 2024/07/17:  coreDNS快速部署

在创建内网服务时，很多设备要指定ip和自定义域名，以及证书，避免每次每台都要修改hosts文件，现在在局域网部署coreDNS

### 拓扑：

```powershell
#环境
ProxMox创建的虚拟环境
使用SDN创建虚拟局域网，IP为192.168.200.10~200
coreDNS ip为192.168.200.20，系统fedora 40 lxc
在Datecenter》SDN》VNets》Subnets，编辑subnet的“DNS Zone Prefix:”为coreDNS的IP

```

### 快速部署

```powershell
## 以下操作在coreDNS设备上操作
#关闭由系统自带的DNS服务
systemctl stop systemd-resolved.service
systemctl disable systemd-resolved
vi /etc/resolv.conf #删除指定的DNS服务
#确认53端口没被占用
lsof -i :53
#下载文件
curl -OL https://github.com/coredns/coredns/releases/download/v1.11.3/coredns_1.11.3_linux_amd64.tgz
#解压文件
tar -xvf coredns_1.11.3_linux_amd64.tgz
#安装
install -c coredns /usr/local/bin
ln -s /usr/local/bin/coredns /usr/bin/coredns
#创建运行用户
useradd coredns -s /sbin/nologin
#创建文件夹
mkdir -vp /etc/coredns/
touch /etc/coredns/Corefile
touch /etc/coredns/hosts
chown -R coredns /etc/coredns/Corefile

```

### 创建systemd文件

```powershell
tee -a /usr/lib/systemd/system/coredns.service <<'EOF'
[Unit]
Description=CoreDNS DNS server
Documentation=https://coredns.io
After=network.target
[Service]
LimitNOFILE=1048576
LimitNPROC=512
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE
PermissionsStartOnly=true
NoNewPrivileges=true
WorkingDirectory=/etc/coredns
User=coredns
ExecStart=/usr/bin/coredns -conf=/etc/coredns/Corefile
ExecReload=/bin/kill -SIGUSR1 $MAINPID
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF
```

### 创建配置文件

```powershell
tee -a /etc/coredns/Corefile <<'EOF'
.:53 {
  # 绑定所有接口
  bind 0.0.0.0
  # hosts 插件: https://coredns.io/plugins/hosts/
  hosts /etc/coredns/hosts {
    # 我们用hosts文件配置ip和域名，方便随时更新
    # 如果有大量自定义域名解析那么建议用file插件使用 符合RFC 1035规范的DNS解析配置文件
    ttl 60
    # 重载hosts配置
    reload 1m
    # 继续执行
    fallthrough
  }
  # file enables serving zone data from an RFC 1035-style master file.
  # 最后所有的都转发到系统配置的上游dns服务器去解析
  forward . 192.168.200.1
  # 缓存时间ttl
  cache 120
  # 自动加载配置文件的间隔时间
  reload 6s
  # 输出日志
  log
  # 输出错误
  errors
}
EOF
```

### 开启服务

```powershell
#加载systemd && 开机启动 && 开启服务 && 查看服务状态
systemctl daemon-reload && systemctl enable coredns && systemctl start coredns && systemctl status coredns
```

如果在内网已经部署了gitlab服务，可以在执行`systemctl status coredns`可以看到有自定义的域名。

如果还未部署，那么现在编辑`/etc/coredns/hosts`，自定义一个域名指向某个IP

```
192.168.200.100 baidu.com
```

### 测试

在其他设备测试DNS服务，在另外一台设备直接ping这个`baidu.com`域名，可以发现解析ip已经指向了指定ip，后续在添加其他服务时，只需要更新coreDNS中的``/etc/coredns/hosts``文件即可。
