# T1195-Supply Chain Compromise-不安全的软件开发周期

## 模块简介

本模块主要是软件开发、发布等生命周期中，由于使用了不安全的编译器、供应商等造成最终软件存在恶意代码。

供应链折衷可以发生在供应链的任何阶段，包括：

- 操纵开发工具
- 操纵开发环境
- 操纵源代码存储库（公共或私有）
- 在开放源代码依赖项中处理源代码
- 操纵软件更新/分发机制
- 受损/感染的系统映像（在工厂中多次感染可移动媒体）
- 用修改后的版本替换合法软件
- 向合法分销商销售经过修改/伪造的产品
- 发布拦截

尽管供应危害会影响硬件或软件的任何组件，但希望获得执行权限的攻击者通常将恶意代码的添加重点放在软件分发或更新渠道中。定位可能是针对特定受害者组或者可能将恶意软件分发给广泛的消费者，但仅针对特定受害者采取了其他策略。在许多应用程序中用作依赖项的流行开源项目也可能有针对性地向依赖项的用户添加恶意代码。

## 检测思路

1. 通过hash或其他完整性检测机制对二进制可执行文件进行验证
2. 监控是否存在恶意调用和恶意链接流量
3. 软件上线前进行安全风险监测
4. 建立基于沙箱的恶意文件检测系统

## 应急手段

1. 开启现有的防病毒网关
2. 安装主机安全的相关产品
3. 开启恶意流量监测系统
4. 更新所有软件至最新
5. 开发过程中使用正版软件

## 排查工具

rpm centos

debsums ubuntu

## 工具操作说明

rpm

```
rpm -Va
```

debsums

```
apt-get install debsums

debsums
sudo debsums --list-missing下一个命令输出没有md5sum信息的文件
```

## 案例说明

暂无