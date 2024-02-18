# 在pc端获取android无线调试的ip和端口

  在使用scrcpy连接Android时，一直有个痛点就是，即使在phone端无线调试是一直开启的，在连接时还是要打开手机看端口，不方便装逼

  在搜索中，发现Android支持mDNS/DNS-SD服务，可以在pc端，通过工具获取局域网内的设备信息，于是可以在pc端，快速查询到设备信息，用以scrcpy连接设备

## linux设备

在linux下获取mDNS/DNS-SD设备信息，需要安装avahi-tools工具

```shell
sudo dnf install avahi-tools
```

  使用：

```shell
#查询adb-tls-connect设备
avahi-browse --terminate --resolve _adb-tls-connect._tcp
```

此时，正常情况下可以看到设备信息已经被扫描出来

```powershell
$ avahi-browse --terminate --resolve _adb-tls-connect._tcp
+ ens160 IPv6 adb-357793090488916-V5NQKL                    _adb-tls-connect._tcp local
+ ens160 IPv4 adb-357793090488916-V5NQKL                    _adb-tls-connect._tcp local
= ens160 IPv4 adb-357793090488916-V5NQKL                    _adb-tls-connect._tcp local
   hostname = [Android-5.local]
   address = [192.168.1.47]
   port = [44683]
   txt = ["v=ADB_SECURE_SERVICE_VERSION"]
= ens160 IPv6 adb-357793090488916-V5NQKL                    _adb-tls-connect._tcp local
   hostname = [Android-5.local]
   address = [192.168.1.47]
   port = [44683]
   txt = ["v=ADB_SECURE_SERVICE_VERSION"]
```

## windows设备

 在windows我找了2个工具获取mDNS/DNS-SD设备信息。一个是FindDevice免安装，但是在使用中发现无法扫描到设备。一个是Nmap，需要安装，可以扫描到设备，但是存在概率扫描不到，需要多扫描几次。

[Nmap网址](https://nmap.org/)

下载后安装，使用参数`nmap --script=broadcast-dns-service-discovery`进行扫描

正常情况会出现扫描设备

```
|     44683/tcp adb-tls-connect
|_      Address=192.168.1.47 fe80::449a:c8ff:fe11:f7f8
```

## 使用scrcpy进行连接

后面就可以使用scrcpy进行连接了

.. tip::
前提是设备已经进行了配对哦😏😏😏😏

```cmd
C:\Users\HJG>scrcpy -w -S --tcpip=192.168.1.47:44683
scrcpy 2.1.1 <https://github.com/Genymobile/scrcpy>
INFO: Connecting to 192.168.1.47:44683...
INFO: Connected to 192.168.1.47:44683
D:\program\scrcpy\scrcpy-server: 1 file pushed, 0 skipped. 7.8 MB/s (56995 bytes in 0.007s)
[server] INFO: Device: [SHARP] SG 808SH (Android 11)
INFO: Renderer: direct3d
INFO: Texture: 1440x3120
[server] ERROR: Failed to start audio capture
[server] ERROR: On Android 11, audio capture must be started in the foreground, make sure that the device is unlocked when starting scrcpy.
WARN: Demuxer 'audio': stream explicitly disabled by the device
[server] INFO: Device screen turned off
```

