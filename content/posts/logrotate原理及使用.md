> logrotate 是一种日志分隔工具，一般在Linux系统发行版中都会自带，它提供了两种分隔解决方案,分别是 create 和copytruncate

#### 一、logrotate原理  
##### create方案

这是logrotate的默认方案，具体的原理是重命名原日志文件然后再创建新的同原名的日志文件（ mv+create ）,在执行完之后，通知应用重新在新文件写入，这几乎是原子操作，所以在日志安全方面更有保障。具体的步骤细节可参照如下：

1.  重命名正在输出日志文件，因为重命名只修改目录以及文件的名称，而进程操作文件使用的是 inode，所以并不影响原程序继续输出日志。
2.  创建新的日志文件，文件名和原日志文件一样，注意，此时只是文件名称一样，而 inode 编号不同，原程序输出的日志还是往原日志文件输出。
3.  最后通过某些方式通知程序，重新打开日志文件；由于重新打开日志文件会用到文件路径而非 inode 编号，所以打开的是新的日志文件。

##### copytruncate方案

在有些情况下比如程序并不支持create的这种方式，就没有提供重新打开日志的接口，重新写入新日志必须要重启应用程序，就必然影响可用性，因此有了copytruncate这种方案

这种方案是把正在输出的日志拷 (copy) 一份出来，然后再清空 (trucate) 原来的日志，清空操作速度还是比较快的，但是如果日志文件太大，那么复制的过程就很漫长了，并且也有可能导致日志丢失。在损失效率和安全的条件下，优点是通用性高，不需要应用程序的支持。细节步骤如下：

1.  将当前正在输出的日志文件复制为目标文件，此时程序仍然将日志输出到原来文件中，此时，原文件名也没有变。
2.  清空日志文件，原程序仍然还是输出到预案日志文件中，因为清空文件只把文件的内容删除了，而 inode 并没改变，后续日志的输出仍然写入该文件中。

#### 二、配置logrotate

> logrotate它只是一个可执行程序，并不是一个服务或者是deemon，所以在进行配置结束后，并不需要重启服务来让配置生效。 

##### 配置文件路径

可执行文件路径： `/usr/sbin/logrotate` 

主配置文件路径: `/etc/logrotate.conf` ，通过 include 指令，会引入 `/etc/logrotate.d` 下的配置文件

自定义配置文件路径: `/etc/logrotate.d/*.conf`，在这个路径下，也有一些第三方软件包自己私有的配置文件。 如 yum，zabbix-agent，syslog，nginx 等。

比较通用的配置文件示例：

```text-plain
/var/log/log_file {
    monthly		#日志文件将按月轮循。其它可用值为daily，weekly或者yearly。
    rotate 5	#一次将存储 5 个归档日志。对于第六个归档，时间最久的归档将被删除。
    dateext		#指示让旧日志文件以创建日期命名
    compress	#在轮循任务完成后，已轮循的归档将使用 gzip 进行压缩。
    delaycompress #与compress 选项一起用，指示logrotate不要将最近的归档压缩，压缩将在下一次轮循周期进行。
    missingok	#在日志轮循期间，任何错误将被忽略，例如 “文件无法找到” 之类的错误。
    notifempty	#如果日志文件为空，轮循不会进行。
    create 644 root root  #以指定的权限创建全新的日志文件，同时 logrotate 也会重命名原始日志文件。
    postrotate  #在所有其它指令完成后，postrotate 和 endscript 里面指定的命令将被执行。
    	/usr/bin/killall -HUP rsyslogd #rsyslogd进程将立即再次读取其配置并继续运行。
    endscript  #在所有其它指令完成后，postrotate 和 endscript 里面指定的命令将被执行。
}
```

常见的配置参数：

```text-plain
daily ：指定转储周期为每天
weekly ：指定转储周期为每周
monthly ：指定转储周期为每月
dateext ：指示让旧日志文件以创建日期命名
rotate count ：指定日志文件删除之前转储的次数，0 指没有备份，5 指保留 5 个备份
tabooext [+] list：让 logrotate 不转储指定扩展名的文件，缺省的扩展名是：.rpm-orig, .rpmsave, v, 和～
missingok：在日志轮循期间，任何错误将被忽略，例如 “文件无法找到” 之类的错误。
size size：当日志文件到达指定的大小时才转储，bytes (缺省) 及 KB (sizek) 或 MB (sizem)
compress： 通过 gzip 压缩转储以后的日志
nocompress： 不压缩
copytruncate：用于还在打开中的日志文件，把当前日志备份并截断
nocopytruncate： 备份日志文件但是不截断
create mode owner group ： 转储文件，使用指定的文件模式创建新的日志文件
nocreate： 不建立新的日志文件
delaycompress： 和 compress 一起使用时，转储的日志文件到下一次转储时才压缩
nodelaycompress： 覆盖 delaycompress 选项，转储同时压缩。
errors address ： 专储时的错误信息发送到指定的 Email 地址
ifempty ：即使是空文件也转储，这个是 logrotate 的缺省选项。
notifempty ：如果是空文件的话，不转储
mail address ： 把转储的日志文件发送到指定的 E-mail 地址
nomail ： 转储时不发送日志文件
olddir directory：储后的日志文件放入指定的目录，必须和当前日志文件在同一个文件系统
noolddir： 转储后的日志文件和当前日志文件放在同一个目录下
prerotate/endscript： 在转储以前需要执行的命令可以放入这个对，这两个关键字必须单独成行
```

#### 三、运行logrotate

##### 命令格式

```text-plain
logrotate [OPTION...] <configfile>
-d, --debug ：debug 模式，测试配置文件是否有错误。
-f, --force ：强制转储文件。
-m, --mail=command ：压缩日志后，发送日志到指定邮箱。
-s, --state=statefile ：使用指定的状态文件。
-v, --verbose ：显示转储过程。
```

##### 定时执行

```text-plain
*/30 * * * * /usr/sbin/logrotate /etc/logrotate.d/rsyslog > /dev/null 2>&1 &
```

##### 手动执行

```text-plain
#debug模式：指定[-d|--debug],并不会真正进行rotate或者compress操作，但是会打印出整个执行的流程，和调用的脚本等详细信息。
logrotate -d <configfile>
#verbose模式：指定[-v|--verbose],会真正执行操作，打印出详细信息（debug 模式，默认是开启verbose）
logrotate -v <configfile>
```