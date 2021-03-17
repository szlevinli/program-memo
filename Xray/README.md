# XRay

## VPS

### BWH

- SPECIAL 20G KVM PROMO V5 - LOS ANGELES - CN2 GIA ECOMMERCE (kind-cubes-1.localdomain)

- IP: 173.242.112.94

- root: YjKjYltBYNxR

- port: 28058

## Install

### Server

- [Xray 一键脚本安装](https://v2xtls.org/xray%e4%b8%80%e9%94%ae%e8%84%9a%e6%9c%ac/)

```bash
bash <(curl -sL https://s.hijk.art/xray.sh)
```

以下是选择了 VLESS+TCP+TLS 安装后的结果

>   Xray运行状态：已安装 Xray正在运行
>
>  Xray配置文件: /usr/local/etc/xray/config.json
>
>  Xray配置信息：
>
>   协议: VLESS
>
>  IP(address): 173.242.112.94
>
>  端口(port)：443
>
>  id(uuid)：ab5818b6-754f-41ec-9ad3-5780f5859eed
>
>  流控(flow)：xtls-rprx-direct
>
>  加密(encryption)： none
>
>  传输协议(network)： tcp
>
>  伪装类型(type)：none
>
>  伪装域名/主机名(host)：www.jl-free.com
>
>  底层安全传输(tls)：XTLS

### Client

- 客户端 MacOS 使用 **QV2Ray**
  - 本地文件: `./Qv2ray.v2.7.0-pre2.macOS-x64.dmg`
  - 下载地址: [QV2Ray](https://github.com/Qv2ray/Qv2ray/releases)
- 客户端 IOS 使用 **Shadowsocket** (*需要美服 APP Store. 具体做法申请新的 AppID, 官网买礼品卡, 更换 IPhone 的购买 AppID. 具体位置在 设置 -> 账户 -> 媒体与购买项目*)

## 日常维护和查看

1. 在 Mac 上键入 `bwg` 登录 server
2. 在 server 上键入 `ixray` 查看 `xray` 的配置信息