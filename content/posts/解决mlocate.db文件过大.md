一、调整mlocate配置文件中将数据目录设置成忽略
vim /etc/updatedb.conf
```
PRUNEPATHS = “/afs /media /net /sfs /tmp /udev /var/cache/ccache /var/spool/cups /var/spool/squid /var/tmp  /data1 /data2 /data3 /data4”
```

二、杀掉当前locate更新任务
```
pkill updatedb.mlocate
```

三、删除历史数据
```
cd /var/lib/mlocate
rm -rf mlocate.*
```

四、手动触发一次更新(稍等片刻)
```
updatedb
```