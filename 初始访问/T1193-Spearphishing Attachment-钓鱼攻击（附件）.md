# T1193-Spearphishing Attachment-钓鱼攻击（附件）

## 模块简介

鱼叉式附件是鱼叉式的特定变体。鱼叉式附件与其他形式的鱼叉式攻击的不同之处在于，它使用了附加到电子邮件的恶意软件。所有形式的鱼叉式攻击都是以电子方式提供的针对特定个人，公司或行业的社会工程。在这种情况下，攻击者会将文件附加到欺骗性电子邮件中，并且通常依靠用户执行来执行。

附件有很多选项，例如Microsoft Office文档，可执行文件，PDF或存档文件。打开附件后，对手的攻击程序会利用漏洞或直接在用户的系统上执行。鱼叉式电子邮件的内容通常带有诈骗的意图，并且可能会解释如何绕开系统保护措施。该电子邮件还可能包含有关如何解密附件（例如zip文件密码）的说明，以逃避电子邮件边界防御。攻击者经常操纵文件扩展名和图标，以使附加的可执行文件看起来像文档文件，或者利用一个应用程序的文件看起来像是另一个文件的文件。

## 检测思路

网络入侵检测系统和电子邮件网关可用于检测传输中带有恶意附件的邮件。

在扫描恶意文档和附件存储在电子邮件服务器或用户计算机上之后，防病毒软件可能会检测到它们。一旦打开附件（例如Microsoft Word文档或PDF到达Internet或生成Powershell.exe），端点感知或网络感知就可能检测到恶意事件，以利用诸如“ 利用客户端执行和脚本开发”之类的技术。

## 应急操作

1. 开启现有的防病毒网关
2. 开启现有的邮件网关
3. 安装主机安全的相关产品
4. 开启恶意流量监测系统

## 排查工具

### 进程类

使用系统自带的进程查看

### 病毒查杀类

chrootkit、rkhunter、clamav

## 工具操作说明

### 进程排查：

Netstat

```
netstat -anp | grep "port"
```

ps

```
ps -aux | grep "/bin/bash"
```

#### 病毒查杀

chkrootkit

网址：[http://www.chkrootkit.org](http://www.chkrootkit.org/)

```
./chkrootkit
```

rkhunter

网址：[http://rkhunter.sourceforge.net](http://rkhunter.sourceforge.net/)

```
rkhunter -c
```

Clamav

ClamAV的官方下载地址为：http://www.clamav.net/download.html

```
#更新病毒库
freshclam
#扫描方法
clamscan -r /etc --max-dir-recursion=5 -l /root/etcclamav.log
clamscan -r /bin --max-dir-recursion=5 -l /root/binclamav.log
clamscan -r /usr --max-dir-recursion=5 -l /root/usrclamav.log
#扫描并杀毒
clamscan -r  --remove  /usr/bin/bsd-port
clamscan -r  --remove  /usr/bin/
clamscan -r --remove  /usr/local/zabbix/sbin
#查看日志发现
cat /root/usrclamav.log |grep FOUND
```

#### 新增文件类

find

```
find / * -perm 777
```

