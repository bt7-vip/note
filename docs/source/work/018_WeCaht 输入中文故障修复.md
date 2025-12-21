
# WeCaht 输入中文故障修复
```{note}
WeChat  AppImage 在Linux 桌面使用一段时间后，会出现无法输入中文的情况，表现就是输入法候选框消失，在其他软件上正常。
```

这种现象会在使用一段时间后出现，或者系统更新后，之前怀疑是WeCaht的原因，在使用新版本之后一段时间又出现相同的问题。今天打算解决这个问题。

在使用WeChat切换输入时尝试从系统日志获取信息，看看有没有打印信息，结果是在系统下没有打印任何信息。然后尝试重启输入法，重启后，问题依旧。

整理这些信息给AI分析，很快给出了结果。是环境变量没有继承。

首先是排查使用的X11还是Wayland，我的环境是Fedora Xfce 42 。目前还是X11。
```shell
echo $XDG_SESSION_TYPE
```

手动指定环境变量然后测试启动WeChat。
```shell
export GTK_IM_MODULE=ibus
export QT_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
./wechatlinux_x86_64.appimage
```

到测试这里输入中文已经正常。结论是：
```{note}
问题的核心是 WeChat AppImage 在启动时没有正确继承或识别系统的输入法环境变量配置
```

修复的话就在桌面图标配置变量参数
```vi
env GTK_IM_MODULE=ibus QT_IM_MODULE=ibus XMODIFIERS=@im=ibus "AppImage Path"
```

在启动命令前添加环境变量，保存后再次登录WeChat，中文输入恢复正常。