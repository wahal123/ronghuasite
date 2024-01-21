### 报错信息：
在正常systemctl启动服务的时候会出现以下信息的报错
```shell
Error getting authority:Eroor initializing authority:Error calling StartServiceByName for org.freedesktop.PolicyKit1:Timeout was readhed(g-io-error-quark, 24)
```
### 原因:
出现这种情况一般是polkit并没有处于正常的激活状态，他会导致其他一些服务也不能正常启动,可以systemctl status polkit 观察以下服务的状态为faild

### 解决方案：
1、直接启动polkit服务 systemctl restart polkit *
2、当发现polkit启动的时候依然出现上述报错，服务启动失败可以尝试以下方法：	
* 重新安装polkit服务 yum install polkit ,然后再进行启动systemctl start polkit 
* 手动启动polkit服务
	* /usr/lib/polkit-1/polkitd --no-debug &
	* systemctl restart dbus.service
	* systemctl start polkit.service*
	