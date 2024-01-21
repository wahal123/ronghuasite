> stress是一个用于施加系统负载的工具，除了 CPU 负载之外，它还可以用来测试其他系统资源，如内存、I/O 和磁盘。
#### 一、指定cpu跑满80%
```shell
# 以下命令的意思是指定80%的cpu核数，然后运行持续60秒
stress --cpu $(echo "$(nproc)*0.8" | bc) --timeout 60
```
#### 二、指定内存跑满80%
```shell
# 以下命令的意思是先算出内存总量的80%，然后指定一个虚拟进程，分配的虚拟进程的大小为内存总量的80%，并且挂起60s
total_mem=$(grep MemTotal /proc/meminfo | awk '{print $2}')
mem_80_percent=$(echo "$total_mem*0.8/1024" | bc)
stress --vm 1 --vm-bytes ${mem_80_percent}M --vm-hang 60

```

