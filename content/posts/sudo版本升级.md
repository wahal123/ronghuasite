1.CentOS/RHEL/Oracle Linux : sudo yum update -y sudo
2.下载不在漏洞影响范围内的安装包，例如 sudo-1.9.5p2。下载地址：https://www.sudo.ws/dist/
解压安装包，进入安装目录，执行以下命令即可：
./configure --prefix=/usr --libexecdir=/usr/lib --with-secure-path --with-all-insults --with-env-editor --docdir=/usr/share/doc/sudo-1.9.5p2 --with-passprompt='sudo password for %p:' && make && make install && ln -sfv libsudo_util.so.0.0.0 /usr/lib/sudo/libsudo_util.so.0

