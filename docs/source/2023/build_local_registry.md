# 搭建本地容器仓库

### 准备工作

安装容器环境

```shell
#安装docker docker-compose
dnf install docker docker-compose
```

### 获取软件

```shell
下载软件
mkdir /deta/{soft,server} -p $$ cd /data/softs
wget https://github.com/goharbor/harbor/releases/download/v2.9.1/harbor-offline-installer-v2.9.1.tgz
解压软件
tar -xvf harbor-offline-installer-v2.9.1.tgz -C /date/server/
cd /data/server/harbor
加载镜像
docker load <harbor.v2.9.1.tar
docker images
```

```shell
cp harbor.yml.tmpl harbor.yml
修改harbor.yml文件
hostanem：
注释https
修改harbor_admin密码
修改data_volume存放位置
```

```shell
./prepare
```

### 开始使用脚本部署harbor服务
```shell
install.sh
```

部署完成后，使用`docker compose ps`查看部署是否成功，如果是成功的，说明harbor部署没问题。

### 将harbor服务使用systemd管理

先将测试部署的harbor关闭

```shell
docker compose down
```

创建service文件`/etc/systemd/system/harbor.service`

```powershell
[Unit]
Description=Harbor
After=docker.service systemd-networkd.service systemd-resolved.service
Requires=docker.service
Documentation=http://github.com/vmware/harbor
[Service]
Type=simple
Restart=on-failure
Restartsec=5
#需要注意narbor的安装位置
ExecStart=docker compose --file /home/syl/harbor/docker-compose.yml up
ExecStop=docker compose --file /home/syl/harbor/docker-compose.yml down
[Install]
WantedBy=multi-user.target
```

```shell
开机启动
systemctl daemon-reload
systemctl enable harbor
systemctl start harbor
```

### 解决podman登录http仓库问题

本地harbor可能没有做https证书签发或者使用的是私有证书，需要在配置私有仓库时允许http或私有证书https

配置文件`/etc/containers/registries.conf`

```yml
#在仓库列别添加私有仓库地址
unqualified-search-registries = ["registry.access.redhat.com", "registry.redhat.io", "docker.io","k8s-register.hjg.com"]
#添加仓库地址
 [[registry]]
prefix = "k8s-register.hjg.com"
location = "192.168.1.130"
insecure = true #此处设置true允许http
```



### 上传镜像

1：公有镜像=> 下载=>重新打标签=>上传到仓库

2：私有镜像

#### podman登录私有仓库

```
podman login h8s-register.hjg.com
```

#### 从公共仓库下载镜像

```
podman pull nginx:latest
```

#### 查询镜像的版本信息

```shell
podman history nginx
```

#### 打标签

```
podman tag nginx:latest k8s-register.hjg.com/hjg/nginx:x.x.x
```

#### 上传到仓库

```
podman push k8s-register.hjg.com/hjg/nginx:1.25.3
```

