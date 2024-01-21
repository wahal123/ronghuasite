**一：重命名网卡配置文件**
这种方法需要修改网卡的配置文件。网卡配置文件位于 `/etc/sysconfig/network-scripts` 目录下，文件名格式为 `ifcfg-<设备名称>`。
1) 使用 `cd` 命令进入 `/etc/sysconfig/network-scripts` 目录。
2) 使用 `mv` 命令将网卡配置文件重命名为您要修改的名称。
例如，要将 `ens33` 网卡的名称修改为 `eth0`，可以使用以下命令：
```bash
cd /etc/sysconfig/network-scripts
mv ifcfg-ens33 ifcfg-eth0
```
3) 修改ifcfg-eth0网络配置文件中的NAME和DEVICE属性
```
sed -i 's/ens33/eth0/g' ifcfg-eth0
```

**二：禁用预定义命名规则**
为防止系统根据预定义的规则进行命名，因此需要在系统启动时传递 `net.ifnames=0 biosdevname=0` 内核参数。
```bash
# 编辑 `/etc/default/grub` 文件
vim /etc/default/grub
# 在 `GRUB_CMDLINE_LINUX` 变量中添加 `net.ifnames=0 biosdevname=0`
GRUB_CMDLINE_LINUX="crashkernel=auto rhgb quiet net.ifnames=0 biosdevname=0"
# 生成新的 grub 配置文件
grub2-mkconfig -o /boot/grub2/grub.cfg
# 重启操作系统
reboot
```
