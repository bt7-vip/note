# windows快速修改远程桌面端口

这是一个powershell脚本，设置端口变量后，执行脚本，即可更改本机的mstsc端口


```powershell
$portvalue = 53892

Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "PortNumber" -Value $portvalue 

New-NetFirewallRule -DisplayName 'RDPPORTLatest-TCP-In' -Profile 'Public' -Direction Inbound -Action Allow -Protocol TCP -LocalPort $portvalue 
New-NetFirewallRule -DisplayName 'RDPPORTLatest-UDP-In' -Profile 'Public' -Direction Inbound -Action Allow -Protocol UDP -LocalPort $portvalue
```
脚本会创建2条防火墙规则，“RDPPORTLatest-TCP-In”和“RDPPORTLatest-UDP-In”，需要跟据实际情况，手动配置端口网络域，是专用还是公用。
确认新端口打开后，重启**Remote Desktop Services**服务使服务监听新端口。
正常情况当前远程桌面会断开，使用新端口重新登录。
配置完成后，建议关闭3389端口。
