# T1091-Replication Through Removable Media-移动介质传播

## 模块简介

通过可移动媒体将恶意软件复制，并在将媒体插入到系统中并执行时利用自动运行功能，攻击者可以进入系统，可能是那些断开连接或存在空隙的网络上的系统。在横向移动的情况下，这可能是通过修改存储在可移动媒体上的可执行文件或通过复制恶意软件并将其重命名为看起来像合法文件来诱使用户在单独的系统上执行它而发生的。在“初始访问”的情况下，这可能是通过手动操作介质，修改用于初始格式化介质的系统或修改介质固件本身而发生的。

## 检测思路

本攻击的检测主要在系统是否允许外部设备接入或者是否有外部设备接入。

## 应急操作

工具类

```
dmidecode 查看系统外设和接口状况
ethtool  查看网卡的相关信息和网线是否连接的状态
dmesg | more 查看硬件信息
lspci 显示外设信息, 如usb，网卡等信息
lsmod 查看已加载的驱动
lshw
```

查看类

```
查看CPU信息：cat /proc/cpuinfo [dmesg | grep -i 'cpu'][dmidecode -t processor]
查看内存信息：cat /proc/meminfo [free -m][vmstat]
查看板卡信息：cat /proc/pci
查看显卡/声卡信息：lspci |grep -i ‘VGA’[dmesg | grep -i 'VGA']
查看网卡信息：dmesg | grep -i ‘eth’[cat /etc/sysconfig/hwconf | grep -i eth][lspci | grep -i 'eth']
查看PCI信息：lspci (相比cat /proc/pci更直观）
查看USB设备：cat /proc/bus/usb/devices
查看键盘和鼠标:cat /proc/bus/input/devices
查看系统硬盘信息和使用情况：fdisk & disk – l & df
查看各设备的中断请求(IRQ):cat /proc/interrupts
```

## 排查工具

使用系统自带命令

## 工具操作说明

参考应急操作

相关案例

暂无