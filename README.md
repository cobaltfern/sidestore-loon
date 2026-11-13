# Sidestore Loon Loopback

用 Loon 替代 LocalDevVPN / StosVPN，让 SideStore 能在 iOS 27 上续签。

SideStore 刷新时连的是 `10.7.0.1:49152`。LocalDevVPN 的作用就是把这个地址反射回手机自己。Loon 从 Build 1007（3.5.3）起，用 `include-tun` + `[IP Rewrite] reflect` 能做到同样的事，不需要再装第二个 VPN。

## 硬性前提

- Loon **Build 1007 / 3.5.3 或更高**。旧版插件可以「启用」，但 IP Rewrite 不生效，也不报错。
- Loon 必须是**手机上的那个**，Mac 版 Loon 的 utun 不属于手机。
- `[IP Rewrite]` **只能在插件里声明**，写进主配置无效；`include-tun` 主配置和插件都可以写。

## 插件内容

见 [`sidestore-loopback.lpx`](sidestore-loopback.lpx)：

```ini
[General]
include-tun = 10.7.0.1/32
ipv6-vif = off

[IP Rewrite]
10.7.0.1 = reflect
```

`include-tun` 只把 SideStore 需要的那一个地址塞进 Loon 的 TUN，不接管其它流量。
`reflect` 的作用是交换源/目的 IP 后回注本机协议栈，端口与载荷保持原样（Loon 官方定义），于是 `10.7.0.1:49152` 落到设备自己的 `remotepairingd` 上。

## 安装

1. Loon → 配置 → 插件 → 右上 `+` → 从 URL 添加，填本仓库的 raw 地址。
2. 导入后确认插件**处于启用状态**（常见假成功：插件在列表里但没勾选）。
3. 打开 Loon 的开关（或 `loon://on`），状态栏出现 VPN 图标。

## SideStore 设置

Settings → ADVANCED SETTINGS → Connection Configuration：

- `Use Local VPN` 打开
- `User Configuration → Device IP` 填 `10.7.0.1`
- `RemotePair Port` 填 `49152`（或留空走自动发现）
- 右上 Confirm → 应出现 "Changes saved"

## 验收

Settings → Health Check，只看两点：

- `Active Protocol` = Remote Pairing
- `10.7.0.1:49152` 可达（testTCP 通过）

然后 My Apps → Refresh All。

注意：**`Tunnel Iface IP` 会显示 Loon 自己 utun 的地址**，不会是 `10.7.0.2/30` —— 那是 LocalDevVPN 的指纹，不是失败标志。

## 常见坑

- Loon 主配置里 `bypass-tun` 通常含 `10.0.0.0/8`，看起来会盖住 `10.7.0.1`。不用改：按 Loon 的优先级规则，掩码不同时范围更小的路由优先，`/32` 会正确生效。
- 插件启用后仍连不上：确认 Loon 的 Build 号（设置里能看到），以及重装插件后是否重新启用。
- 依旧失败时，回退到 App Store 的 [LocalDevVPN](https://apps.apple.com/app/localdevvpn/id6755608044)，或 fork 版 `LocalDevVPN-custom.ipa`（修了 iOS 26.4+ 的 /32 对端路由）。

## 依据

- Loon 插件文档（`[IP Rewrite]` 动作定义）：https://nsloon.app/docs/Plugin/
- Loon 通用配置（`include-tun`）：https://nsloon.app/docs/General/#include-tun
- 同思路的 Surge 方案：https://bssn520.github.io/p/22
