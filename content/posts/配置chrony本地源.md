chrony时间服务器分为两个部分:chronyd和chronyc
##### 局域网源服务端：chronyd
* vim /etc/chrony.conf
```
#注释掉原来的所有public server，并使用本机作为时间源
server 192.168.231.131 iburst

#这里可以修改同步时最大单步时间变动范围，以下的配置表示每次同步时客户端与服务端相差一小时内直接切换，否则同步时最大只能改变1h,多余的时间误差等待下次同步（时间变动过大可能会导致不可预测的问题）。默认为1s。
makestep 3600 10

#打开server连接失败时使用本地时间，层数最高为10
local stratum 10
```


##### 局域网源客户端：chronyd
vim /etc/chrony.conf
```
# 该参数可以多次用于添加时钟服务器，必须以"server"格式使用，一般而言，你想添加多少服务器，就可以添加多少服务器。
server 0.rhel.pool.ntp.org iburst
server 1.rhel.pool.ntp.org iburst
server 2.rhel.pool.ntp.org iburst
server 3.rhel.pool.ntp.org iburst

# stratumweight指令设置当chronyd从可用源中选择同步源时，每个层应该添加多少距离到同步距离。默认情况下，设置为0，让chronyd在选择源时忽略源的层级。
stratumweight 0

# chronyd程序的主要行为之一，就是根据实际时间计算出计算机增减时间的比率，将它记录到一个文件中是最合理的，它会在重启后为系统时钟作出补偿，甚至可能的话，会从时钟服务器获得较好的估值。
driftfile /var/lib/chrony/drift

# rtcsync指令将启用一个内核模式，在该模式中，系统时间每11分钟会拷贝到实时时钟（RTC）。
rtcsync

# 通常，chronyd将根据需求通过减慢或加速时钟，使得系统逐步纠正所有时间偏差。在某些特定情况下，系统时钟可能会漂移过快，导致该调整过程消耗很长的时间来纠正系统时钟。该指令强制chronyd在调整期大于某个阀值时步进调整系统时钟，但只有在因为chronyd启动时间超过指定限制（可使用负值来禁用限制），没有更多时钟更新时才生效。
makestep 10 3

# 这里你可以指定一台主机、子网，或者网络以允许或拒绝NTP连接到扮演时钟服务器的机器。
#allow 192.168.56.6
#deny 192.168/16

# 该指令允许你限制chronyd监听哪个网络接口的命令包（由chronyc执行）。该指令通过cmddeny机制提供了一个除上述限制以外可用的额外的访问控制等级。
bindcmdaddress 127.0.0.1
bindcmdaddress ::1

# Serve time even if not synchronized to any NTP server.

#local stratum 10

keyfile /etc/chrony.keys

# Specify the key used as password for chronyc.

commandkey 1

# Generate command key if missing.

generatecommandkey

# Disable logging of client accesses.

noclientlog

# Send a message to syslog if a clock adjustment is larger than 0.5 seconds.

logchange 0.5

logdir /var/log/chrony

#log measurements statistics tracking
```

##### 管理工具：chronyc
```shell
# 显示系统时钟与NTP服务器之间的偏差，并提供有关时钟稳定性的信息
chronyc tracking
# 显示已配置的NTP服务器列表及其状态
chronyc sources
# 强制时钟进行一次大步调整，以立即将时钟与NTP服务器同步
chronyc -a makestep
# 显示NTP服务器的统计信息，包括延迟、偏差和抖动
chronyc sourcestats
# 将Chrony设置为在线状态，并开始与NTP服务器同步
chronyc online
# 将Chrony设置为离线状态，并停止与NTP服务器同步
chronyc offline



```
