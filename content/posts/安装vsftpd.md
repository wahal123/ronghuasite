### 一、安装vsftpd服务器程序

在线安装或者下载rpm包上传到无网环境的机器

```text-plain
yum install -y vsftpd
#设置为开机启动
systemctl enable vsftpd
```

编译安装

```text-plain
 略
```

###  二、建立宿主用户

```text-plain
useradd ftp
mkdir /data/ftp -p
usermod -d /data/ftp -g ftp -s /sbin/nologin ftp
echo "XF8C84hXfEqGX8dG" |passwd ftp --stdin
touch /var/log/vsftpd.log
chown -R ftp.ftp /var/log/vsftpd.log
touch /etc/vsftpd/chroot_list

#需要注意的几点
#1、因为ftp指定了nologin，而且sftp在安装完成后会有设置允许哪个shell登录的pam认证机制，所以要对nologin进行配置为shell
	vim /etc/shells
	/sbin/nologin
#2、新版本的vsftp增强了安全机制，必须设置其家目录没有写入权限。所以在指定的家目录去掉写权限
	mkdir -p /data/ftp/public
	chown ftp.ftp -R /data/ftp
	chmod a-w /data/ftp
```

### 三、修改配置文件

vsftpd的配置文件路径：/etc/vsftpd/vsftpd.conf，作为运维工作者应该知道的，在修改配置文件前，一定要先备份完后再进行动作，主要修改的配置文件如下：

```text-plain
anonymous_enable=YES
local_enable=YES
write_enable=YES
local_umask=022
idle_session_timeout=600
data_connection_timeout=1200
anon_upload_enable=NO
anon_mkdir_write_enable=NO
dirmessage_enable=YES

connect_from_port_20=YES
chown_uploads=NO

#开启日志审计
xferlog_enable=YES
xferlog_file=/var/log/vsftpd.log
xferlog_std_format=YES

nopriv_user=ftp
async_abor_enable=YES
ascii_upload_enable=YES
ascii_download_enable=YES
ftpd_banner=Welcome to blah FTP service.
chroot_local_user=YES
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd/chroot_list
ls_recurse_enable=NO
listen=YES
pam_service_name=vsftpd
userlist_enable=YES
#tcp_wrappers=YES
guest_enable=YES
guest_username=ftp
virtual_use_local_privs=YES
user_config_dir=/etc/vsftpd/vconf.d
```

以上参数的具体解释：

```text-plain
anonymous_enable=NO //不允许匿名用户访问。
local_enable=YES //设定本地用户可以访问。
write_enable=YES //设定可以进行写操作。
local_umask=022 //设定上传后文件的权限掩码。
idle_session_timeout=600
data_connection_timeout=1200 //设置超时时间
anon_upload_enable=NO //禁止匿名用户上传。
anon_mkdir_write_enable=NO //禁止匿名用户建立目录。
dirmessage_enable=YES ///设定开启目录标语功能。
xferlog_enable=YES ///设定开启日志记录功能。
connect_from_port_20=YES ///设定端口20进行数据连接。
chown_uploads=NO ///设定禁止上传文件更改宿主。
xferlog_file=/var/log/vsftpd.log ///设定Vsftpd的服务日志保存路径。
xferlog_std_format=YES ///设定日志使用标准的记录格式。
nopriv_user=ftp ///设定支撑Vsftpd服务的宿主用户为手动建立的Vsftpd用户。
注意，一旦做出更改宿主用户后，必须注意一起与该服务相关的读写文件的读写赋权问题。比如日志文件就必须给与该用户写入权限等。

async_abor_enable=YES ///设定支持异步传输功能。
ascii_upload_enable=YES
ascii_download_enable=YES ///设定支持ASCII模式的上传和下载功能。
ftpd_banner=Welcome to blah FTP service.//设定Vsftpd的登陆标语
chroot_local_user=YES
chroot_list_enable=YES ///禁止用户登出自己的FTP主目录。

chroot_list_file=/etc/vsftpd/chroot_list ///如果开启了chroot_list_enable=YES，那么一定要开启这个，这条是锁定登录用户只能家目录的位置。
注：建立chroot_list文件
touch/etc/vsftp/chroot_list，然后将帐户输入一行一个，保存就可以了，如果不需要限制用户，也可以只建立一个空文件，或者将
chroot_list_enable=NO

ls_recurse_enable=NO ///禁止用户登陆FTP后使用"ls -R"的命令。该命令会对服务器性能造成巨大开销。如果该项被允许，那么挡多用户同时使用该命令时将会对该服务器造成威胁。

listen=YES ///设定该Vsftpd服务工作在StandAlone模式下。顺便展开说明一下，所谓StandAlone模式就是该服务拥有自己的守护进程支持，在ps -A命令下我们将可用看到vsftpd的守护进程名。如果不想工作在StandAlone模式下，则可以选择SuperDaemon模式，在该模式下 vsftpd将没有自己的守护进程，而是由超级守护进程Xinetd全权代理，与此同时，Vsftp服务的许多功能将得不到实现。
pam_service_name=vsftpd ///设定PAM服务下Vsftpd的验证配置文件名。因此，PAM验证将参考/etc/pam.d/下的vsftpd文件配置。
userlist_enable=YES ///设定userlist_file中的用户将不得使用FTP。
tcp_wrappers=YES ///设定支持TCP Wrappers。

使用虚拟用户需要增加以下部分默认中不包含这些设定项目，需要自己手动添加:
guest_enable=YES ///设定启用虚拟用户功能。
guest_username=ftp ///指定虚拟用户的宿主用户。
virtual_use_local_privs=YES ///设定虚拟用户的权限符合他们的宿主用户。
user_config_dir=/etc/vsftpd/vconf.d ///指定用户Vsftp的配置文件存放路径，这里的用户可为虚拟用户可为主机系统用户

这个被指定的目录里，将存放每个Vsftp虚拟用户个性的配置文件，注:就是这些配置文件名必须和虚拟用户名相同。
```

### 四、添加vsftpd虚拟用户

#### 4.1 创建虚拟用户文本文件

虚拟用户文本文件中写入所要创建的虚拟用户名和密码，奇数行为用户名，偶数行为密码。文件放置在目录`/etc/vsftpd/vconf.d`中

```text-plain
mkdir /etc/vsftpd/vconf.d
vim vuser.txt
ftpuser1
ftpuser1password
```

#### 4.2 生成db数据库文件

将之前创建的虚拟用户密码文件生成为db文件，权限置为600

```text-plain
db_load -T -t hash -f vuser.txt vuser.db
chmod 600 vuser.db
```
注意：在较新的操作系统中的pam权限因为更改了生成db文件的格式，因为具体是哪个版本更改的不清楚，所以要根据报错情况来选择使用哪种方式，另外一种生成的方式如下：
```
# 生成用户名密码数据文件
gdbmtool -n user_new.pag store ftpuser1 ftpuser1password
# 查看对应的数据库文件某个用户的密码
gdbmtool user_new.pag fetch  ftpuser1
```

#### 4.3 在pam中配置通过数据库认证

```text-plain
vim /etc/pam.d/vsftpd
#%PAM-1.0
auth sufficient pam_userdb.so db=/etc/vsftpd/vconf.d/vuser
account sufficient pam_userdb.so db=/etc/vsftpd/vconf.d/vuser
session    optional     pam_keyinit.so    force revoke
auth       required     pam_listfile.so item=user sense=deny file=/etc/vsftpd/ftpusers onerr=succeed
auth       required     pam_shells.so
#auth       required    pam_nologin.so
auth       include      password-auth
account    include      password-auth
session    required     pam_loginuid.so
session    include      password-auth
```

#### 4.4 添加虚拟账户配置文件

虚拟账户没有对应的配置文件不会生效，在vsftpd.conf文件中确定了虚拟账户的配置文件目录为：`/etc/vsftpd/vconf.d`

```text-plain

cd /etc/vsftpd/vconf.d
#根据先前配置的虚拟用户根据用户名创建文件
touch ftpuser1

#为每个用户配置相应的用户权限等
vim ftpuser1
local_root=/data/ftpuser/
write_enable=YES
download_enable=YES
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
anon_world_readable_only=NO

#创建用户目录
mkdir -p /data1/ftp/ftpuser1
chmod a-w /data1/ftp/ftpuser1



---------------------字段说明--------------------------------
1、完全控制配置文件


local_root=/data/ftp/ftpuser1 //虚拟用户的个人目录路径
anon_world_readable_only=NO
//如果开启,那么所有非匿名登陆的用户名都会被切换成guest_username指定的用户名

anon_upload_enable=YES //匿名用户可以上传

anon_mkdir_write_enable=YES //匿名用户可以建目录

anon_other_write_enable=YES //匿名用户其它的写权利

local_max_rate=1048576 //本地用户的最大传输速度，单位是Byts/s

2、只能下载无其他权限
local_root=/data/ftp/ftpuser1 //虚拟用户的个人目录路径

write_enable=NO //用户无写权限
anon_world_readable_only=NO

anon_upload_enable=NO //匿名用户不可以上传

anon_mkdir_write_enable= NO //匿名用户不可以建目录

anon_other_write_enable= NO //匿名用户无写权利

local_max_rate=1048576 //本地用户的最大传输速度，单位是Byts/s

-----------------------------------------------------------
```

### 五、开放ftp防火墙端口

### 六、重启vsftpd