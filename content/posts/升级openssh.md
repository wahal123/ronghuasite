### 主要步骤：
	1、下载官方最新的openssh安装包
	2、编译成rpm
	3、备份原来的配置文件
	4、开始使用rpm包进行升级

### 一、下载官方openssh包
进入官网https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/，然后找到最新的安装包[openssh-9.3p2.tar.gz](https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-9.3p2.tar.gz)

### 二、制作RPM包
```bash
# 创建两个目录SOURCES和SPECS
mkdir -p /usr/src/redhat/{SOURCES,SPECS}
# 安装包上传
将官网下载的安装包openssh-9.3p2.tar.gz上传到目录/usr/src/redhat/SOURCES/中
# 提取openssh.spec文件
tar -zxvf openssh-9.3p2.tar.gz openssh-9.3p2/contrib/redhat/openssh.spec
# 拷贝文件openssh.spec到 SPECS 目录
cp openssh-9.3p2/contrib/redhat/openssh.spec ../SPECS/
# 变更文件openssh.spec属组和属组为sshd
cd /usr/src/redhat/SPECS && chown sshd:sshd openssh.spec
# 备份openssh.spec文件
cp openssh.spec{,.bak}
# 关闭no_gnome_askpass no_x11_askpass这两个参数
sed -i -e "s/%global no_x11_askpass 0/%global no_x11_askpass 1/g" openssh.spec
sed -i -e "s/%global no_gnome_askpass 0/%global no_gnome_askpass 1/g" openssh.spec
# 注释BuildRequires
vim openssh.spec
注释这一行BuildRequires: openssl-devel < 1.1

# 创建预编译目录
mkdir -p  ~/rpmbuild/SOURCES/
cp /usr/src/redhat/SOURCES/openssh-9.3p2.tar.gz ~/rpmbuild/SOURCES/

# 下载x11-ssh-askpass文件并放入~/rpmbuild/SOURCES/文件夹中
wget https://src.fedoraproject.org/repo/pkgs/openssh/x11-ssh-askpass-1.2.4.1.tar.gz/8f2e41f3f7eaa8543a2440454637f3c3/x11-ssh-askpass-1.2.4.1.tar.gz

# 安装依赖
yum install rpm-build gcc make wget openssl-devel krb5-devel pam-devel libX11-devel xmkmf libXt-devel gtk2-devel openldap-devel
# 执行构建rpm
cd /usr/src/redhat/SPECS/
rpmbuild -ba openssh.spec
# 查看编译后的目录结构
tree -L 2 /root/rpmbuild/
生成的rpm包在/root/rpmbuild/RPMS/x86_64下

```

### 三、备份原版本的openssh
```bash
# 1. 创建备份目录
mkdir /root/sshbackup
# 2. 将/etc/ssh目录下的所有文件压缩备份到/root/sshbackup目录
cd /root/sshbackup && tar -zcvf sshdbak.tgz -C /etc/ssh .
# 3. 将/etc/pam.d/sshd文件备份到/root/sshbackup目录，升级完成后还需要恢复
mkdir /root/sshbackup/pam.d && cp -p  /etc/pam.d/{sshd,system-auth} /root/sshbackup/pam.d

```

### 四、开始升级
```bash
# 1. 开始进行升级
rpm -Uvh /root/rpmbuild/RPMS/x86_64/*.rpm
# 2. 开始恢复pam.d的文档
## 恢复前先把新的pam.d文件备份
cp -p /etc/pam.d/sshd{,_afterupgrade} 
cp -p /etc/pam.d/system-auth{,_afterupgrade}
## 进行恢复pam.d文件
cp -p /root/sshbackup/pam.d/* /etc/pam.d/
# 3.恢复sshd配置文件
tar -zxvf sshdbak.tgz -C /etc/ssh
# 4.配置公私钥的权限
chmod 600 /etc/ssh/ssh_host_*
# 5.配置ssh允许root用户登录，如果不需要可不进行变更
sed -i 's/^#\(PermitRootLogin\)/\1/g' /etc/ssh/sshd_config
# 6. 重启sshd服务
systemctl restart sshd
```

### 五、升级脚本
```bash
SSH_BACKDIR=/root/sshbackup  
RPM_PACKAGEDIR=/root/openssh_upgrade  
  
upgrade_toversion=$(ls $RPM_PACKAGEDIR|head -1 |awk -F- '{print $2}')  
current_version=$(ssh -V 2>&1 |awk -F_ '{print $2}'|awk -F, '{print $1}')  
#创建备份目录  
if [ ! -e $SSH_BACKDIR ];then  
mkdir -p $SSH_BACKDIR  
fi  
#备份etc配置文件  
tar -zcvf $SSH_BACKDIR/sshdbak.tgz -C /etc/ssh .  
  
#备份PAM文件  
if [ ! -e $SSH_BACKDIR/pam.d ];then  
mkdir -p $SSH_BACKDIR/pam.d  
fi  
cp -p /etc/pam.d/{sshd,system-auth} $SSH_BACKDIR/pam.d  
  
if [ $(ls $SSH_BACKDIR/pam.d 2>/dev/null|wc -l) -eq 2 ] && [ -e $SSH_BACKDIR/sshdbak.tgz ];then  
if [ "$upgrade_toversion" == "$current_version" ];then  
echo "版本已升级"  
exit  
else  
#开始升级  
rpm -Uvh $RPM_PACKAGEDIR/*.rpm  
  
#备份升级后的PAM配置文件  
cp -p /etc/pam.d/sshd{,_afterupgrade}  
cp -p /etc/pam.d/system-auth{,_afterupgrade}  
  
#开始从备份文件恢复PAM  
cp -p $SSH_BACKDIR/pam.d/* /etc/pam.d/  
  
#开始恢复ETC配置文件并给公私钥变更权限  
tar -zxvf $SSH_BACKDIR/sshdbak.tgz -C /etc/ssh  
chmod 600 /etc/ssh/ssh_host_*  
  
#配置ssh允许root用户登录，如果不需要可不进行变更  
sed -i 's/^#\(PermitRootLogin\)/\1/g' /etc/ssh/sshd_config  
fi  
fi
```


