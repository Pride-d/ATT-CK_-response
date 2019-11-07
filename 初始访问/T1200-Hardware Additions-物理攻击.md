# T1200-Hardware Additions-物理攻击

## 模块简介

对手可能会将计算机附件，计算机或网络硬件引入系统或网络中，这些系统或网络可以用作获取访问权限的媒介。尽管很少有APT团体公开使用的参考资料，但许多渗透测试人员还是利用硬件的增加来进行初始访问。商业和开放源代码产品具有以下功能：无源网络窃听，中间人加密中断，键盘监控，通过DMA读取内核内存[[4\]](https://www.youtube.com/watch?v=fXthwl6ShOg)以及向现有网络等。

## 检测思路

该攻击手法主要在内网中接入恶意设备进行攻击，故主要检测思路为恶意流量发现和内网资产识别。

## 应急操作

1. 设置内部服务器认证白名单
2. 开启防火墙或交换机开启IP/MAC绑定
3. 禁止某IP的网络请求
4. 恶意流量监测产品的日志审计

## 排查工具

### 流量类

wireshark、Tcpdump

### 请求类

#### iptables(Centos7&7一下)

```
#允许对外请求的返回包
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
#允许icmp包通过
iptables -A INPUT -p icmp --icmp-type any -j ACCEPT
#允许来自于lo接口的数据包，如果没有此规则，将不能通过127.0.0.1访问本地服务
iptables -A INPUT -i lo -j ACCEPT

#常用端口
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 21 -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 23 -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 8080 -j ACCEPT

#过滤所有非以上规则的请求
iptables -P INPUT DROP

#如果要添加内网ip信任（接受其所有TCP请求）
#注：(**.**.**.**)为IP,下同
iptables -A INPUT -p tcp -s **.**.**.** -j ACCEPT

#要封停一个IP，使用下面这条命令
iptables -I INPUT -s **.**.**.** -j DROP
#要解封一个IP，使用下面这条命令
iptables -D INPUT -s **.**.**.** -j DROP
#SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
#HTTP
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 8080 -j ACCEPT
#HTTPS
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
#POP3
iptables -A INPUT -p tcp --dport 110 -j ACCEPT
#SMTP
iptables -A INPUT -p tcp --dport 25 -j ACCEPT
#FTP
iptables -A INPUT -p tcp --dport 21 -j ACCEPT
iptables -A INPUT -p tcp --dport 20 -j ACCEPT
#DNS
iptables -A INPUT -p tcp --dport 53 -j ACCEPT
3.添加使用IP限制INPUT访问规则,这里拿SSH为例,192.168.0.100为允许的IP
代码如下复制代码
#DELETE
iptables -D INPUT -p tcp --dport 22 -j ACCEPT
#ADD
iptables -A INPUT -s 192.168.0.100 -p tcp --dport 22 -j ACCEPT
```

#### Firewall-cmd(Centos 7)

```
转自https://havee.me/linux/2015-01/using-firewalls-on-centos-7.html
一、介绍
防火墙守护 firewalld 服务引入了一个信任级别的概念来管理与之相关联的连接与接口。它支持 ipv4 与 ipv6，并支持网桥，采用 firewall-cmd (command) 或 firewall-config (gui) 来动态的管理 kernel netfilter 的临时或永久的接口规则，并实时生效而无需重启服务。

zone
Firewall 能将不同的网络连接归类到不同的信任级别，Zone 提供了以下几个级别

drop: 丢弃所有进入的包，而不给出任何响应
block: 拒绝所有外部发起的连接，允许内部发起的连接
public: 允许指定的进入连接
external: 同上，对伪装的进入连接，一般用于路由转发
dmz: 允许受限制的进入连接
work: 允许受信任的计算机被限制的进入连接，类似 workgroup
home: 同上，类似 homegroup
internal: 同上，范围针对所有互联网用户
trusted: 信任所有连接
过滤规则
source: 根据源地址过滤
interface: 根据网卡过滤
service: 根据服务名过滤
port: 根据端口过滤
icmp-block: icmp 报文过滤，按照 icmp 类型配置
masquerade: ip 地址伪装
forward-port: 端口转发
rule: 自定义规则
其中，过滤规则的优先级遵循如下顺序

source
interface
firewalld.conf
二、使用方法
# systemctl start firewalld         # 启动,
# systemctl enable firewalld        # 开机启动
# systemctl stop firewalld          # 关闭
# systemctl disable firewalld       # 取消开机启动
具体的规则管理，可以使用 firewall-cmd，具体的使用方法可以

$ firewall-cmd --help

--zone=NAME                         # 指定 zone
--permanent                         # 永久修改，--reload 后生效
--timeout=seconds                   # 持续效果，到期后自动移除，用于调试，不能与 --permanent 同时使用
1. 查看规则
查看运行状态

$ firewall-cmd --state
查看已被激活的 Zone 信息

$ firewall-cmd --get-active-zones
public
  interfaces: eth0 eth1
查看指定接口的 Zone 信息

$ firewall-cmd --get-zone-of-interface=eth0
public
查看指定级别的接口

$ firewall-cmd --zone=public --list-interfaces
eth0
查看指定级别的所有信息，譬如 public

$ firewall-cmd --zone=public --list-all
public (default, active)
  interfaces: eth0
  sources:
  services: dhcpv6-client http ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
查看所有级别被允许的信息

$ firewall-cmd --get-service
查看重启后所有 Zones 级别中被允许的服务，即永久放行的服务

$ firewall-cmd --get-service --permanent
2. 管理规则
# firewall-cmd --panic-on           # 丢弃
# firewall-cmd --panic-off          # 取消丢弃
# firewall-cmd --query-panic        # 查看丢弃状态
# firewall-cmd --reload             # 更新规则，不重启服务
# firewall-cmd --complete-reload    # 更新规则，重启服务
添加某接口至某信任等级，譬如添加 eth0 至 public，永久修改

# firewall-cmd --zone=public --add-interface=eth0 --permanent
设置 public 为默认的信任级别

# firewall-cmd --set-default-zone=public
a. 管理端口
列出 dmz 级别的被允许的进入端口

# firewall-cmd --zone=dmz --list-ports
允许 tcp 端口 8080 至 dmz 级别

# firewall-cmd --zone=dmz --add-port=8080/tcp
允许某范围的 udp 端口至 public 级别，并永久生效

# firewall-cmd --zone=public --add-port=5060-5059/udp --permanent
b. 网卡接口
列出 public zone 所有网卡

# firewall-cmd --zone=public --list-interfaces
将 eth0 添加至 public zone，永久

# firewall-cmd --zone=public --permanent --add-interface=eth0
eth0 存在与 public zone，将该网卡添加至 work zone，并将之从 public zone 中删除

# firewall-cmd --zone=work --permanent --change-interface=eth0
删除 public zone 中的 eth0，永久

# firewall-cmd --zone=public --permanent --remove-interface=eth0
c. 管理服务
添加 smtp 服务至 work zone

# firewall-cmd --zone=work --add-service=smtp
移除 work zone 中的 smtp 服务

# firewall-cmd --zone=work --remove-service=smtp
d. 配置 external zone 中的 ip 地址伪装
查看

# firewall-cmd --zone=external --query-masquerade
打开伪装

# firewall-cmd --zone=external --add-masquerade
关闭伪装

# firewall-cmd --zone=external --remove-masquerade
e. 配置 public zone 的端口转发
要打开端口转发，则需要先

# firewall-cmd --zone=public --add-masquerade
然后转发 tcp 22 端口至 3753

# firewall-cmd --zone=public --add-forward-port=port=22:proto=tcp:toport=3753
转发 22 端口数据至另一个 ip 的相同端口上

# firewall-cmd --zone=public --add-forward-port=port=22:proto=tcp:toaddr=192.168.1.100
转发 22 端口数据至另一 ip 的 2055 端口上

# firewall-cmd --zone=public --add-forward-port=port=22:proto=tcp:toport=2055:toaddr=192.168.1.100
f. 配置 public zone 的 icmp
查看所有支持的 icmp 类型

# firewall-cmd --get-icmptypes
destination-unreachable echo-reply echo-request parameter-problem redirect router-advertisement router-solicitation source-quench time-exceeded
列出

# firewall-cmd --zone=public --list-icmp-blocks
添加 echo-request 屏蔽

# firewall-cmd --zone=public --add-icmp-block=echo-request [--timeout=seconds]
移除 echo-reply 屏蔽

# firewall-cmd --zone=public --remove-icmp-block=echo-reply
g. IP 封禁
# firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='222.222.222.222' reject"
当然，我们仍然可以通过 ipset 来封禁 ip

封禁 ip

# firewall-cmd --permanent --zone=public --new-ipset=blacklist --type=hash:ip
# firewall-cmd --permanent --zone=public --ipset=blacklist --add-entry=222.222.222.222
封禁网段

# firewall-cmd --permanent --zone=public --new-ipset=blacklist --type=hash:net
# firewall-cmd --permanent --zone=public --ipset=blacklist --add-entry=222.222.222.0/24
导入 ipset 的 blacklist 规则

# firewall-cmd --permanent --zone=public --new-ipset-from-file=/path/blacklist.xml
如果已经存 blacklist，则需要先删除

# firewall-cmd --get-ipsets
blacklist
# firewall-cmd --permanent --zone=public --delete-ipset=blacklist
然后封禁 blacklist

# firewall-cmd --permanent --zone=public --add-rich-rule='rule source ipset=blacklist drop'
重新载入以生效

# firewall-cmd --reload
查看 blacklist

# firewall-cmd --ipset=blacklist --get-entries
```

