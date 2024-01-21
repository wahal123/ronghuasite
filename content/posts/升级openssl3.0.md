### 1、下载官方源码包

下载地址：https://www.openssl.org/source/

### 2、编译openssl前的准备

1.  安装zlib库：`yum install zlib zlib-dev`
2.  安装IPC/Cmd.pm模块：  
    `yum install perl-CPAN`  
    `perl -MCPAN -e shell`，根据shell的提示，按照默认的配置进行配置，此过程时间稍长，看到cpan[1]标识后，表示配置完成  
    `install IPC/Cmd.pm`  
      
     

### 3、对下载的源码包进行编译

```text-plain
tar -zxvf openssl-3.0.3.tar.gz && cd openssl-3.0.3
./config --prefix=/usr/local/openssl --openssldir=/usr/local/opensslconf
make 
make test
make install 
```

### 4、对老版本的openssl进行备份

为防止新版本openssl带来的新问题，在升级前需要对现有的openssl进行备份,主要有3个地方

```text-plain
#上传编译好的包到/usr/local/下
rz openssl_3.0.7_amd64.compiled.tgz

#备份
mv  /usr/bin/openssl{,_bak202211}
mv /usr/lib64/openssl{,_bak202211}
mv /usr/include/openssl{,_bak202211}
```

### 5、将新版本的openssl库文件链接到对应位置

```text-plain
ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl
ln -s /usr/local/openssl/lib64/ /usr/lib64/openssl
ln -s /usr/local/openssl/include/openssl/ /usr/include/openssl
```

### 6、加载系统变量

```text-plain
#不执行此步骤会提示 “libssl.so.3: cannot open shared”，这是因为在当前的主机环境变量的动态库搜索路径没有配置/usr/lib64,可以将其加入到系统变量中
echo /usr/lib64/openssl >/etc/ld.so.conf.d/openssl.conf
ldconfig -v
```

### 7、查看版本

```text-plain
[root@ronghua openssl]# openssl version 
OpenSSL 3.0.3 3 May 2022 (Library: OpenSSL 3.0.3 3 May 2022)
 
```

注意：如果在执行上述步骤后查看版本，，比如在.bashrc中配置`export LD_LIBRARY_PATH =/usr/lib64`，还有一种方式是使用ldconfig命令，配置路径到ldconfig的配置文件中，让其进行加载，比如，

### 8、脚本处理

```text-plain
backup(){
if [ -e "/usr/bin/openssl" ];then
	mv  /usr/bin/openssl{,_bak202205}
	
fi
if [ -e "/usr/lib64/openssl" ];then
	mv /usr/lib64/openssl{,_bak202205}
fi
if [ -e "/usr/include/openssl" ];then
	mv /usr/include/openssl{,_bak202205}
fi
}

generate_update(){
ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl
ln -s /usr/local/openssl/lib64/ /usr/lib64/openssl
ln -s /usr/local/openssl/include/openssl/ /usr/include/openssl
}

add_lib(){
echo "/usr/lib64/openssl" >/etc/ld.so.conf.d/openssl.conf
ldconfig -v
}

recovery(){
rm -f /usr/bin/openssl
rm -f /usr/lib64/openssl
rm -f /usr/include/openssl

if [ -e "/usr/bin/openssl_bak202205" ];then
	mv  /usr/bin/openssl_bak202205 /usr/bin/openssl
fi
if [ -e "/usr/lib64/openssl_bak202205" ];then
	mv /usr/lib64/openssl_bak202205 /usr/lib64/openssl
fi
if [ -e "/usr/include/openssl_bak202205" ];then
	mv /usr/include/openssl_bak202205 /usr/include/openssl
fi
}

upgrade(){
backup
generate_update
add_lib
}
if [ $1 == "recorvery" ];then
	recovery
elif [ $# -eq 0 ];then
	upgrade
fi
```