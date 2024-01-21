### 一、安装配额工具(如果已安装跳过此步)

```text-plain
[root@localhost ~]# yum -y install quota
```

### 二、查看当前配额是否开启

```text-plain
mount 
/dev/sda3 on / type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
/dev/sda1 on /boot type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
```

### 三、开启配额

针对 quota 限制的项目主要有三项，如下所示：

-   uquota/usrquota/quota： 针对用户账号的设定
-   gquota/grpquota：            针对群组的设定
-   pquota/prjquota：              针对单一目录的设定

命令格式： mount -o uquota,gquota [设备]  [挂载点]

**注意：grpquota和prjquota不能同时存在！！！在下列的命令行中均以“|”加以区分**

```text-plain
#对磁盘进行重新挂载(先卸载)
[root@localhost ~]# umount  /dev/sdb1
[root@localhost ~]# mount -o uquota，gquota|pquota /dev/sdb1 /data
[root@localhost ~]# mount |grep quota
/dev/sda3 on / type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
/dev/sda1 on /boot type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
/dev/sdb1 on /data type xfs (rw,relatime,seclabel,attr2,inode64,usrquota,grpquota)
```

### 四、开机自动开启配额并挂载

```text-plain
[root@localhost ~]# echo '/dev/sdb1 /data xfs defaults,usrquota,grpquota|pquota 0 0'>> /etc/fstab
[root@localhost ~]# tail -1 /etc/fstab
/dev/sdb1 /data xfs defaults,usrquota,grpquota 0 0
```

### 五、设置配额

命令格式：xfs_quota -x -c “limit [-ug] b[soft|hard]=N i[soft|hard]=N ”   [挂载点]  
命令格式：xfs_quota -x -c “limit [-up] b[soft|hard]=N i[soft|hard]=N ”   [挂载点]

#### 5.1 针对用户和组级别的限制

```text-plain
-x            使用配额模式，只有此模式才能设置配额
-c            启用命令模式
report        显示配额信息
limit         设置配额
bsoft         软限制磁盘大小(硬盘)
bhard         硬配额磁盘大小
isoft		  软限制文件个数
ihard	   	  硬限制文件个数
-u            用户
-g            组
-p 			  针对项目，比如可以对目录进行限制

#限制磁盘soft 10M，hard为100M，文件软限制个数为10个，硬限制为100个，用户为ronghua（注意这里不能为root,否则限制会不生效）
[root@localhost ~]# xfs_quota -x -c 'limit bsoft=10M bhard=100M isoft=10 ihard=100 -u ronghua' /data
[root@localhost ~]# chmod 777 /data
```

#### 5.2 针对project的限制(也就是目录的限制，eg:/app/myquota)

```text-plain
##绑定项目ID和目录和项目名称的对应关系
echo "1:/app/myquota" > /etc/projects  #设置项目ID和目录的对应关系(绑定ID和目录)
echo "myquotaproject:1" > /etc/projid  #设置项目名称和项目ID的对应关系(绑定名称和ID)
xfs_quota -x -c "project -s myquotaproject"  #初始化项目名称

##对目录设定具体的限制值
xfs_quota -x -c "limit  bsoft=450M bhard=500M -p myquotaproject" /app
```

### 六、查看配额信息

print：单纯的列出目前主机内的档案系统参数等数据  
df：与原本的df一样的功能，可以加上-b（block）-i（inode）-h（加上单位）等  
report：列出目前的quota项目，有-ugr（user/group/project）及-bi等数据  
state：说明目前支持quota的档案系统的信息，有没有起动相关项目等

命令格式：xfs_quota -x -c ‘report’  [挂载点]

```text-plain
[root@localhost ~]# xfs_quota -x -c 'report' /data
User quota on /data (/dev/sdb1)
                               Blocks                     
User ID          Used       Soft       Hard    Warn/Grace     
---------- -------------------------------------------------- 
ronghua                4      10240     102400     00 [--------]

Group quota on /data (/dev/sdb1)
                               Blocks                     
Group ID         Used       Soft       Hard    Warn/Grace     
---------- -------------------------------------------------- 
ronghua                4          0          0     00 [--------]

```

### 七、查看状态信息

```text-plain
[ronghua@localhost data]# xfs_quota -x -c 'print'
Filesystem          Pathname
/                   /dev/sda3
/boot               /dev/sda1
/data               /dev/sdb1 (uquota, gquota)
```

### 八、验证

```text-plain
#先验证文件个数限制的情况
[ronghua@localhost data]#for i in {1..12};do dd bs=1M count=1 if=/dev/zero of=$i.txt;done
dd: failed to open ‘11.txt’: Disk quota exceeded
dd: failed to open ‘12.txt’: Disk quota exceeded

#再验证文件大小的情况
[ronghua@localhost data]#dd bs=200M count=1 if=/dev/zero of=1.txt 
dd: error writing ‘1.txt’: Disk quota exceeded

[ronghua@localhost data]#ll 1.txt -h
total 100M
-rw-r--r--. 1 ronghua ronghua 100M Jun 16 12:13 1.txt
```