当使用只读变量进行配置后，我们发现再使用常规的unset 命令却无法进行取消了。下面是使用gdb的方式将其只读变量取消，主要的测试过程。

```text-plain
#首先设置一个只读变量
[root@localhost ~]# readonly aaa=1

#只读变量设置后，当我们不需要了，试着取消之前设置的变量，发现报错了
[root@localhost ~]# unset aaa
-bash: unset: aaa: cannot unset: readonly variable

#接下来使用gdb调试工具对当前bash进行调试处理，查询当前终端运行的bash进程号
[root@localhost ~]# echo $$
34896

#对该进程号进行调试
[root@localhost ~]# gdb --pid 34896
GNU gdb (GDB) Red Hat Enterprise Linux 7.6.1-120.el7
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Attaching to process 34896
Reading symbols from /usr/bin/bash...Reading symbols from /usr/bin/bash...(no debugging symbols found)...done.
(no debugging symbols found)...done.
Reading symbols from /lib64/libtinfo.so.5...Reading symbols from /lib64/libtinfo.so.5...(no debugging symbols found)...done.
(no debugging symbols found)...done.
Loaded symbols for /lib64/libtinfo.so.5
Reading symbols from /lib64/libdl.so.2...(no debugging symbols found)...done.
Loaded symbols for /lib64/libdl.so.2
Reading symbols from /lib64/libc.so.6...(no debugging symbols found)...done.
Loaded symbols for /lib64/libc.so.6
Reading symbols from /lib64/ld-linux-x86-64.so.2...(no debugging symbols found)...done.
Loaded symbols for /lib64/ld-linux-x86-64.so.2
Reading symbols from /lib64/libnss_files.so.2...(no debugging symbols found)...done.
Loaded symbols for /lib64/libnss_files.so.2
0x00007f73eafa646c in waitpid () from /lib64/libc.so.6
Missing separate debuginfos, use: debuginfo-install bash-4.2.46-34.el7.x86_64
#这部分是已经进入gdb调试bash窗口，我们调用取消变量绑定函数，注意，参数变量必须指定为双引号，单引号会报错
(gdb) call unbind_variable("aaa")
$1 = 0
#取消以后可以直接敲q进行退出，退出前会提示是否要求剥离调试进程号，选择是
A debugging session is active.
	Inferior 1 [process 34896] will be detached.
Quit anyway? (y or n) y

#然后再对只读变量进程操作，发现可以进行设置了
[root@localhost ~]# unset aaa
[root@localhost ~]# 
```

以上部分是具体的部分说明，在互联网上看到的更多的是几行命令复制粘贴到终端执行就可以了，这只是换了一种方式，其实是将需要执行的操作管道送给gdb，比如以下部分

```text-plain
cat << EOF |gdb		
attach $$
call unbind_variable("aaa")
detach
EOF
```