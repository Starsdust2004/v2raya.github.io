---
title: "OpenWrt"
description: "安装内核和 v2rayA"
lead: "v2rayA 的功能依赖于 V2Ray 内核，因此需要安装内核。"
date: 2020-11-16T13:59:39+01:00
lastmod: 2020-11-16T13:59:39+01:00
draft: false
images: []
menu:
  docs:
    parent: "installation"
weight: 15
toc: true
---

## 通过 v2rayA 自建软件源安装

参考：

1. [v2rayA for OpenWrt 仓库主页](https://github.com/v2raya/v2raya-openwrt)

2. [OSDN 主页](https://osdn.net/projects/v2raya/)

可以使用反向代理了 OSDN 的开源镜像站来加速下载。

## 通过 OpenWrt 官方软件源安装

{{% notice info %}}
目前只有 openWrt 最新的 snapshot 版本软件源中含有 v2rayA，使用此版本的用户可以直接从软件源安装。
{{% /notice %}}

```bash
opkg update
opkg install v2raya
```

{{% notice note %}}
由于目前 openWrt 的软件源中没有 `v2ray-core`, `xray-core` 会作为依赖被安装。
如果你打算使用 v2ray，那么你需要手动安装它。在同时存在 v2ray 与 xray 的情况下，v2rayA 将优先使用前者。
{{% /notice %}}

## 手动安装

### 安装 V2Ray 内核 / Xray 内核

首先安装软件包 `unzip` 与 `wget`，然后从 [Github Releases](https://github.com/v2fly/v2ray-core/releases) 下载 v2ray 内核然后将其保存到 `/usr/bin`，最后给予二进制文件可执行权限。

例如：

```bash
opkg update; opkg install unzip wget-ssl
wget https://github.com/v2fly/v2ray-core/releases/download/v4.40.1/v2ray-linux-64.zip
unzip -d v2ray-core v2ray-linux-64.zip
cp v2ray-core/v2ray v2ray-core/v2ctl /usr/bin
chmod +x /usr/bin/v2ray; chmod +x /usr/bin/v2ctl
```

{{% notice note %}} **擦亮眼睛**
格外注意你的 OpenWrt 设备的架构，不要下载到不适用于你设备的版本，否则内核将无法运行。
{{% /notice %}}

### 安装 v2rayA

{{% notice info %}}
对于软件源中没有 v2rayA 的用户，可以从 [此处](https://downloads.openwrt.org/snapshots/packages) 中寻找适合你架构的 ipk 文件进行安装，也可以直接按如下方式手动安装。
{{% /notice %}}

从 [Github Releases](https://github.com/v2rayA/v2rayA/releases) 下载最新版本对应处理器架构的二进制文件，然后移动到 `/usr/bin` 并给予执行权限：

```bash
wget https://github.com/v2rayA/v2rayA/releases/download/v$version/v2raya_linux_$arch_$version --output-document v2raya
mv v2raya /usr/bin/v2raya && chmod +x /usr/bin/v2raya
```

### 安装依赖包与内核模块

```bash
opkg update
opkg install \
    ca-bundle \
    ip-full \
    iptables-mod-conntrack-extra \
    iptables-mod-extra \
    iptables-mod-filter \
    iptables-mod-tproxy \
    kmod-ipt-nat6
```

### 创建配置文件和服务文件

`/etc/config/v2raya` 参考[此处](https://raw.githubusercontent.com/openwrt/packages/master/net/v2raya/files/v2raya.config)。

`/etc/init.d/v2raya` 参考[此处](https://raw.githubusercontent.com/openwrt/packages/master/net/v2raya/files/v2raya.init)。

给予此文件可执行权限：

```bash
chmod +x /etc/init.d/v2raya
```

## 运行

### 开启 v2rayA 服务

```bash
uci set v2raya.config.enabled='1'
uci commit v2raya
```

### 启动 v2rayA

```bash
/etc/init.d/v2raya start
```

## 常见故障

### PPPoE 拨号问题

如果你通过 PPPoE 拨号上网，那么你可能会遇到 v2rayA 的透明代理开启一段时间后没有网络连接的故障。解决方法是，使用 v2rayA 的时候不要删除或替换“网络 > 接口”默认的 WAN 连接（该连接使用 DHCP 协议），而应该新建一个接口来进行拨号。**新建的 PPPoE 拨号接口需要添加到名为 wan 的防火墙区域。**

### 防止DNS污染对局域网设备不生效

编辑 "接口 -> LAN -> 使用自定义的 DNS 服务器" 为 "127.2.0.17" 即可让局域网内的其他设备也享受到 "防止DNS污染" 的效果

### 部分设备无法运行

~~v2rayA 所用的[数据库模块](https://github.com/boltdb/bolt)目前不支持基于 MIPS 的芯片，这部分设备（比如一些便宜的 WiFi 路由器、国产龙芯电脑等）可能无法正确初始化数据库，从而导致无法使用。~~
该问题已经于 v1.5.9.1698.1 版本得到解决。

另外，如果设备闪存空间太小，v2rayA 也会无法启用。有需要的朋友可以用 `upx` 压缩 v2rayA 与核心再试试。

内核模块不全的操作系统无法开启透明代理，建议使用 OpenWrt 官方发行分支或者 ImmortalWrt 第三方分支。
