/etc/pam.d/sshd （远程ssh）

/etc/pam.d/login (终端)

在第一行下即#%PAM-1.0的下面添加:  
auth required pam_tally2.so deny=3 unlock_time=600 even_deny_root root_unlock_time=1200

各参数解释:  
even_deny_root： 也限制root用户；

deny ：设置普通用户和root用户连续错误登陆的最大次数，超过最大次数，则锁定该用户

unlock_time： 设定普通用户锁定后，多少时间后解锁，单位是秒；

root_unlock_time： 设定root用户锁定后，多少时间后解锁，单位是秒；

手动解除锁定：  
查看某一用户错误登陆次数：  
pam_tally --user  
例如，查看work用户的错误登陆次数：  
pam_tally --user work  
清空某一用户错误登陆次数：  
pam_tally --user --reset  
例如，清空 work 用户的错误登陆次数，  
pam_tally --user work –-reset

如果使用pam_tally没生效的话，也可以使用pam_tally2命令：

pam_tally2 --u tom --reset将用户的计数器重置清零（SLES 11.2和12版本下用此命令才重置成功）

查看错误登录次数：pam_tally2 --u tom

faillog -r 命令清空所有用户错误登录次数  
在服务器端以root用户登录  
执行命令：  
# faillog –a 查看用户登录错误次数

faillog -u user –r 清空指定用户user的错误登录次数

```text-plain
  如果超过三次的话，用户不能登录并且此后登录用户错误登录次数还是会增加。
  在登录错误次数不满三次时，登录成功后，则这个用户登录错误值将清零，退出后重新telnet登录将采用新的计数。
```

其他例子：  
Pam_tally2锁定SSH登录

默认情况下，pam_tally2模块已经安装在大多数Linux发行版，它是由PAM包本身的控制。 本文演示如何锁定和深远的登录尝试的失败一定次数后解锁SSH帐户。

如何锁定和解锁用户帐户  
使用“/etc/pam.d/password-auth”配置文件来配置的登录尝试的访问。 打开此文件并以下AUTH配置行举行的“ 身份验证 ”部分的开头添加到它。

auth required pam_tally2.so file=/var/log/tallylog deny=3 even_deny_root unlock_time=1200  
接下来，添加以下行“ 账户 ”部分。

account required pam_tally2.so  
参数  
文件= /无功/日志/ tallylog -默认的日志文件是用来保持登录计数。  
否认= 3 -拒绝后，3次尝试访问和锁定用户。  
even_deny_root -政策也适用于root用户。  
unlock_time = 1200 -帐户将被锁定，直到20分钟 。 （如果要永久锁定，直到手动解锁，请删除此参数。）  
一旦你使用上面的配置完成，现在尽量尝试使用任何“ 用户名 ”3失败的登录尝试到服务器。 当你取得了超过3次，你会收到以下消息。

[root@test01 ~]# ssh test01@172.16.25.126  
test01@172.16.25.126’s password:  
Permission denied, please try again.  
test01@172.16.25.126’s password:  
Permission denied, please try again.  
test01@172.16.25.126’s password:  
Account locked due to 4 failed logins  
Account locked due to 5 failed logins  
Last login: Mon Apr 22 21:21:06 2017 from 172.16.16.52  
现在，使用以下命令验证或检查用户尝试的计数器。

[root@test01 ~]# pam_tally2 --user=test01  
Login Failures Latest failure From  
test01 15 04/22/17 21:22:37 172.16.16.52  
如何重置或解锁用户帐户以再次启用访问。

[root@test01 pam.d]# pam_tally2 --user=test01 --reset  
Login Failures Latest failure From  
test01 15 04/22/13 17:10:42 172.16.16.52  
验证登录尝试已重置或解锁

[root@test01 pam.d]# pam_tally2 --user=test01  
Login Failures Latest failure From  
test01 0  
PAM模块是所有Linux发行版中都有的， 在命令行中执行“ 人pam_tally2”可更多地了解它。