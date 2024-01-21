### 封堵端口只允许本地访问

-   将INPUT链上所有的ip都拒绝掉
-   开放INPUT链上源地址为本机ip为允许

```text-plain
#因为防火墙的执行顺序是从后往前的，也就是iptables -L上看到的大数字先执行，为了使所有ip都没有遗漏，所以要将规则放置在最后
>$ iptables -A INPUT -ptcp --dport 10250 -j DROP

#关于允许本机访问，我试了将本地环回地址127.0.0.1用于-s下开放，后来发现配置这个ip后，所有ip就都能访问了，不是仅仅只允许127.0.0.1进行访问，所以后来想到的策略是只允许本机的地址，如本机地址为192.168.0.224
>$ iptables -I INPUT -ptcp -s 192.168.0.224 --dport 10250 -j ACCEPT
```

### 实现本地端口转发

#### 1、开启ipv4转发

```text-plain
vim /etc/sysctl.conf
net.ipv4.ip_forward = 1
sysctl -p
```

#### 2、在PREROUTING链实现转发

```text-plain
#例如访问端口2222然后转发给本机22端口，这种场景在不改变sshd配置文件，且不开放22端口时使用,比较常用
iptables -t nat -A PREROUTING -p tcp --dport 2222 -j REDIRECT --to-ports 22
```

#### 3、端口映射

```text-plain
#通常可以将内网的端口暴露给公网使用，即端口映射，如：将内网224的8080端口映射给公网124.71.62.103的8080端口
iptables -t nat -A PREROUTING -p tcp -d 124.71.62.103 --dport 8080 -j DNAT --to-destination 192.168.0.224:8080
iptables -t nat -A POSTROUTING -p tcp -s 192.168.0.224 --sport 8080 -j SNAT --to-destination 124.71.62.103:8080 
```

#### 4、远程端口转换

```text-plain
#例如使用192.168.0.139:5555去访问192.168.0.239:9999端口，那么需要在192.168.0.139的机器上执行下面的命令
iptables -t nat -A PREROUTING -p tcp -d 192.168.0.139 --dport 5555 -j DNAT --to-destination 192.168.0.239:9999
iptables -t nat -A POSTROUTING -p tcp -d 192.168.0.239 --dport 9999 -j SNAT --to-source 192.169.0.139  
```

### 指定开放多端口多ip访问

```text-plain
#iptables 选项 链名称 -m 扩展模块 --具体扩展条件 -j 动作
#一次需要过滤或开放很多端口
iptables  -A  INPUT  -p tcp  -m  multiport --dports  20,80,20000:22000  -j  ACCEPT

#根据IP地址范围开放端口
iptables  -A  INPUT  -p tcp  --dport  22  -m  iprange  --src-range  192.168.1.10-192.168.1.20   -j  ACCEPT
或
iptables -A INPUT -p tcp --dport 22  -s 192.168.1.0/24  -j  ACCEPT
```

### 禁止访问外网

```text-plain
#先设置能够访问的ip段
iptables -I OUTPUT -d 192.168.231.0/24 -j ACCEPT
#禁用所有出去的网络
iptables -A OUTPUT -j DROP
```

将本机的
```shell
iptables -t nat -A OUTPUT -p tcp -d 43.136.36.97 --dport 13032 -j DNAT --to-destination 172.16.0.17:30018
```