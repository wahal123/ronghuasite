卡型号：NVIDIA Corporation GP100GL [Tesla P100 PCIe 12GB] (rev a1)
### 一、ESXI配置
如果centos是虚拟机时，需要esxi相应配置
	1、pcie卡使用为直通模式，同时启动时应该使用uefi加载启动
			配置pcie的启动模式为uefi时，应该在宿主机的bios里面进行配置，一般情况下我们在改变uefi启动模式时只会修改相应的bios加载启动，而忽略pcie加载的引导方式，在bios里的pcie选项中的所有都调整为efi引导方式。
	2、exis创建虚拟机centos时应该使用EFI引导安装，编辑设置--> 虚拟机选项-->引导选项-->固件-->EFI，注意这里不要启动安全引导，如图所示：
	{{< figure src="/images/Pasted image 20230403144305.png" caption="">}}
	3、内存配置应该预留所有客户机内存，编辑设置--> 虚拟机硬件-->内存下拉-->启用“预留所有客户机内存”
	{{< figure src="/images/Pasted image 20230403144537.png" caption="">}}
	4、增加内存映射相关配置。编辑设置--> 虚拟机选项-->高级-->配置参数-->编辑配置，增加pciPassthru.use64bitMMIO=“TRUE”，pciPassthru.64bitMMIOSizeGB=32G(这个值配置时应该是显卡显存数*2，然后向上匹配选择最接近2的n次幂的值)
	{{< figure src="/images/Pasted image 20230403145332.png" caption="">}}
	

### 二、安装GPU驱动
以上述的配置进行安装centos7，安装完成以后，开始准备安装GPU驱动
相应的准备工作：
#### 2.1 安装相应的编译辅助程序包
```shell
yum -y install epel-release
yum -y install gcc binutils wget
yum -y install kernel-devel

```
#### 2.2 关闭图形化启动页面以及nouveau
```shell
# 关闭图形化页面
systemctl set-default multi-user.target

# 关闭nouveau
	# 修改nouveau配置
	echo -e "blacklist nouveau\noptions nouveau modeset=0" > /etc/modprobe.d/blacklist.conf
	# 备份系统镜像
	mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak
	# 重建系统镜像
	dracut /boot/initramfs-$(uname -r).img $(uname -r)
	# 重启系统
	reboot
	# 检查nouveau是否已经被关闭，如果没有任何输出那么就已经关闭了
	lsmod | grep nouveau

```
#### 2.3 开始安装GPU启动
##### 2.3.1 首先找到GPU的型号
```shell
[root@localhost ~]# lspci|grep -i nvidia
03:00.0 3D controller: NVIDIA Corporation GP100GL [Tesla P100 PCIe 12GB] (rev a1)
0b:00.0 3D controller: NVIDIA Corporation GP100GL [Tesla P100 PCIe 12GB] (rev a1)
```
##### 2.3.2 下载NVIDIA驱动
去官网网站下载驱动，官方站点的地址为：https://www.nvidia.cn/Download/index.aspx?lang=cn，以下的图片是根据型号筛选出来的所需要安装的驱动。
{{< figure src="/images/Pasted image 20230403152356.png" caption="">}}
将上述的驱动包下载完以后，上传至centos服务器，并给与可执行权限chmod +x NVIDIA-Linux-x86_64-525.105.17.run。同时在上述的类别中我们发现这个包依赖于CUDA Toolkit,因此在安装驱动之前首先应先安装CUDA。
##### 2.3.3 安装CUDA
首先选择登录官网下载CUDA的包，根据网站上的导航项筛选出对应的操作系统，架构以及安装的类型，这里我们选择使用rpm(local)的方式，之后会弹出相应的安装步骤，根据步骤一步步操作即可，以下的命令可以作为参考：
```shell
wget https://developer.download.nvidia.com/compute/cuda/12.1.0/local_installers/cuda-repo-rhel7-12-1-local-12.1.0_530.30.02-1.x86_64.rpm
rpm -ivh cuda-repo-rhel7-12-1-local-12.1.0_530.30.02-1.x86_64.rpm
yum clean all
yum -y install nvidia-driver-latest-dkms
yum -y install cuda
```
##### 2.3.4 安装NVIDIA驱动
```shell
# 安装依赖包
dnf install -y tar bzip2 make automake gcc gcc-c++ pciutils elfutils-libelf-devel libglvnd-devel
# 直接执行驱动包然后按照步骤按顺序执行
[root@localhost ~]# ./NVIDIA-Linux-x86_64-525.105.17.run
# 安装dkms
dnf install dkms
```
以下部分是截图的安装顺序
继续安装
{{< figure src="/images/Pasted image 20230403153742.png" caption="">}}
会进入安装的过程，等待中
{{< figure src="/images/Pasted image 20230403153828.png" caption="">}}
提示安装32位兼容库，这里可以选则不安装
{{< figure src="/images/Pasted image 20230403153920.png" caption="">}}
当这里出现安装失败的时候，可以按照图片提示的命令进行强制安装，/usr/sbin/dkms install --no-depmod -m nvidia -v 525.105.17 -k 3.10.0-1160.el7.x86_64 --force
{{< figure src="/images/Pasted image 20230403154143.png" caption="">}}
##### 2.3.5 查看安装是否成功
```shell
# 直接在终端中输入以下命令，如有响应并且显示nvidia的相关信息则表示安装成功，如下所示
[root@localhost ~]# nvidia-smi
Mon Apr  3 23:59:20 2023       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 525.105.17   Driver Version: 525.105.17   CUDA Version: 12.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Tesla P100-PCIE...  Off  | 00000000:03:00.0 Off |                    0 |
| N/A   35C    P0    31W / 250W |      0MiB / 12288MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   1  Tesla P100-PCIE...  Off  | 00000000:0B:00.0 Off |                    0 |
| N/A   34C    P0    27W / 250W |      0MiB / 12288MiB |      2%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

以上的安装文档是参照互联网不同的安装步骤并且经实践且能安装成功总结而来，很多我其实并不太能理解具体的作用，为什么这么配，由于时间关系这些细节并没有沉下去去细究，这里主要作为一个总结存档，万一以后再需要安装的时候，能够照着步骤一步步执行，以完成任务为优先