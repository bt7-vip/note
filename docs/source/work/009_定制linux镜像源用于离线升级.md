# 定制linux镜像rpm源用于离线升级

在运维时，会遇到处于生产环境的设备，需要升级软件包或者升级系统，等保要求比较高的企业会记录所有入场软件信息，比如刻盘才能带进机房，这种情况下，将整个iso镜像刻盘，时间消耗很多。这时就面临到一个问题：`我们需要一个离线源，源只包含我们用到的依赖包，并且使用yum或apt-get自动解决依赖，体积尽可能小，方便快速复制。`

制作源需要两个关键信息。1：包。2：包的关系。

## 一：下载包

以下演示在fedora下操作，redhat/centos系统均适用

准备一台和生产环境相同的系统环境，版本和软件安装情况尽量一致，这个在进场前就咨询好准确信息，后面升级就会很顺利

#### 方法一：

```shell
#创建文件夹
sudo mkdir ~/rpm/Packages
#依据当前系统环境下载包，已安装的包不下载
sudo dnf update XXXX --downloadonly --downloaddir=~/rpm/Packages/
```

如果只知道少量信息，不确定那些关键包的版本，或者为了稳妥一点，可以使用下载全量依赖包的方法，将所有依赖包准备好

#### 方法二：

```shell
#创建文件夹
sudo mkdir ~/rpm/Packages
#安装下载工具
sudo dnf install dnf-utils
#开始下载
cd ~/rpm/Packages
sudo repotrack XXX
```

全量依赖包相比部分依赖体积要大上不少，但是相比与几个G的镜像体积，还算很好的，刻盘也很快。等级要求比较高的建议使用方法二，减少流程，更加稳妥

## 二：创建关联信息

包下载好后，就可以创建依赖信息，挂载后让yum/dnf自动解决依赖

```shell
#安装createrepo
sudo dnf install createrepo_c
#创建xml关联文件
sudo createrepo ~/rpm/
#创建后会在~/rpm/下生成一个repodata文件夹，rpm包信息及相关依赖关系会在文件中记录
```

创建完成后，将rpm文件夹刻盘带进机房，在离线环境，将源指向rpm文件夹，先`dnf clean all `再`dnf makecache`就可以使用源了。

### FAQ

升级软件包，要定清楚要做那些操作，如果你要做一些编译动作，都要把这些操作需要的软件准备，防止生产环境没有gcc的情况，不过不建议在生产环境直接编译，最好模环境进打包好进生产环境直接升级，防止你升级依赖导致服务对某个包只兼容指定版本，造成服务启动失败。

--处处是坑处处坑
