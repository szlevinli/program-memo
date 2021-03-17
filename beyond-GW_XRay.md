# XRay

## VPS

### BWH

- SPECIAL 20G KVM PROMO V5 - LOS ANGELES - CN2 GIA ECOMMERCE (kind-cubes-1.localdomain)

- IP: 173.242.112.94

- root: YjKjYltBYNxR

- port: 28058

## Install

- [Xray 一键脚本安装](https://v2xtls.org/xray%e4%b8%80%e9%94%ae%e8%84%9a%e6%9c%ac/)

```bash
bash <(curl -sL https://s.hijk.art/xray.sh)
```

以下是选择了 VLESS+TCP+TLS 安装后的结果

>  安装Xray...
>
>  Xray最新版 v1.4.0 已经安装
>
>  BBR模块已安装
>
>  Xray启动成功
>
> 
>
>  Xray运行状态：已安装 Xray正在运行
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
>  id(uuid)：b64d9688-ca67-47ab-8fa9-8081d237645d
>
>  流控(flow)：无
>
>  加密(encryption)： none
>
>  传输协议(network)： tcp
>
>  伪装类型(type)：none
>
>  伪装域名/主机名(host)：www.jl-free.com
>
>  底层安全传输(tls)：TLS

- 客户端 MacOS 使用 [QV2Ray](https://github.com/Qv2ray/Qv2ray/releases)
- 客户端 IOS 使用

## 日常维护和查看

1. 在 Mac 上键入 `bwg` 登录 server
2. 在 server 上键入 `ixray` 查看 `xray` 的配置信息