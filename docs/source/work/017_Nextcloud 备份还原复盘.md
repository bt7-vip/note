##  Nextcloud 备份还原复盘

```{note}
也许你会疑惑，一个备份还原怎么还需要复盘
  我确认我需要复盘，因为这件事我花了三天才做完。这是痛苦的三天，直到我编写复盘文档时，还在跑还原
```
###  起因

  oracle 平台提供的镜像，系统盘规划的空间只能安装nextcloud几个容器，我在打开在线文档时，跟目录(/)直接塞满。第一次部署nextcloud是使用的外挂存储卷，并且在一次虚拟机故障后，跟目录(/)没有多余空间运行容器。在探索过可以使用不同发行版的系统后，决定使用rocky或者fedora来运行nextcloud，初始镜像在更新系统后还不到1G，可以将数据和系统放在一个一个50G的块。

###  测试不同发行版

####  导入镜像

大致流程如下：
- 将qcow2镜像下载到本地
- 创建一个存储桶
- 上传镜像到存储桶中
- 在实例侧边栏最下方有个定制映像
- 导入映像，选择从桶中导入

####  创建实例或者替换引导卷

导入后的定制映像就可以拿来使用，直接通过映像创建新实例或者将运行的实例替换引导卷达到换系统的目的

替换引导卷使用指定映像，从列表选择就可以直接选择前面导入的映像
还有一种是指定引导卷，这种对系统有要求，需要替换同发行版的引导卷

在替换时，在高级选项中添加 metadata 条目 ssh_authorized_keys，值为公钥，以便通过 SSH 密钥登录

之后就可以替换。我是没找到怎么指定用户名，所以使用的是发行版默认的用户名称，所以我用了fedora。

fedora： fedora

替换完开机后就可以使用密钥ssh登录

####  登录系统
---
首次替换的是rocky发行版，但是替换后无法登录，网也不通，所以就换成了fedora 。
因为fedora是用的新建实例，一次就登录成功，我又新建创建了rocky，问题依旧。创建时没找到配置用户和密码的方法，所以也无法从控制台登录。等于被锁在外面。所以我又换成了fedora。

####  vnc链接到系统

在实例操作系统管理 > 控制台连接，这里可以创建一个vnc连接到虚拟机的console界面。

```{note}
vnc提供的ssh是rsa，新的发行版有些不在支持rsa，在创建vnc连接时，建议下载key，避免兼容问题。
```

我在本地安装了一个rocky8 desktop专门用来登录vnc
在创建本地连接后，复制linux/mac的vnc连接，这个连接。然后在连接的两个ssh后面，添加 -i ssh.key 来指定私钥，在shell中执行。

运行后就可以通过系统自带的vnc连接本地的5900端口了。
在vnc里可以折腾的更多了，但是我发现有些系统进单用户模式失败。界面显示“display output is not active”， 这个问题没有解决，所以前面进不去系统在这里就放弃了，所以我又换了fedora。

###  新系统部署nextcloud

安装fail2ban，配置防火墙，安装dokcer，为了避免麻烦，这次使用root运行的doker
其实已经运行了一个redhat 10虚拟机，但是在这个系统上没有成功运行fail2ban，暂时放在那让别人扫了。%% 所以我又换了fedora %%。

根据ai的建议，首先部署nextcloud aio
```shell
sudo docker run --init --sig-proxy=false --name nextcloud-aio-mastercontainer --restart always --env NEXTCLOUD_MEMORY_LIMIT=4096M --publish 80:80 --publish 8080:8080 --publish 8443:8443 --volume nextcloud_aio_mastercontainer:/mnt/docker-aio-config --volume /var/run/docker.sock:/var/run/docker.sock:ro  nextcloud/all-in-one:latest
```

其实一开始我是将块直接挂在到这个实例上，想要直接通过备份恢复，结果恢复到后来log一直显示没有找到打包文件，但是备份文件其实就是在那个位置，最终放弃，将备份的系统恢复，重启拉起旧系统从旧系统备份**迁移**。

####  还原（迁移）nextcloud
虚拟机没有桌面，需要将端口转发到本地
```shell
 ssh -i id_ed25519 -L 8080:localhost:8080 user@ip -p 端口
```
这样本地浏览器就可以打开`https://127.0.0.1:8080`来管理
第一次打开会给你密码，记住密码，进入后不要验证域名，那样会进入新nextcloud部署，选择下面的从备份恢复。


这里需要在新宿主机安装borgbackup工具，配置新到旧实例的免密，然后用borg测试备份是否正确识别到,备份目录权限建议777,恢复后删除旧实例备份。
```shell
borg list --format '{archive}{NL}' ssh://usr@ip:port/mnt/backup1/borg/
```

出现备份列表后，表示borg正常，直接将密钥对复制到borg容器的目录下，让borgbackup使用这个密钥登录旧机


```shell
##  查询backup volume
sudo docker volume ls
##  查看文件位置
sudo docker volume inspect nextcloud_aio_mastercontainer
##  复制私钥替换公钥
sudo cp .ssh/id_rsa /var/lib/docker/volumes/nextcloud_aio_mastercontainer/_data/data/id_borg
##  根据私钥生成公钥
sudo ssh-keygen -y -f /var/lib/docker/volumes/nextcloud_aio_mastercontainer/_data/data/id_borg |sudo tee /var/lib/docker/volumes/nextcloud_aio_mastercontainer/_data/data/id_borg.pub
```

这时可以根据前面测试的路径，再次用aio测试备份，正常情况应该可以发现备份。我在这里卡了1天。正确的逻辑就是，新机的容器可以直接免密登录旧宿主机。

然后就是恢复备份，等待即可。

###  遇到的坑

1: borg从远程备份，正常来说都用户做免密就可以访问，但是最后还是用测试好的密钥替换容器生成密钥才能登录。

2:  旧机是用备份卷拉起的新实例，用着用着把我ip给墙了，真tm草了。

3: 还原后的新设备，重新解析域名后，需要等待一段时间让更新证书，不要着急一直刷新。

4:  部署容器时使用变量指定容器的内存，默认内存给512m，打开一个网页要2分钟加载完，心态早就炸了。
