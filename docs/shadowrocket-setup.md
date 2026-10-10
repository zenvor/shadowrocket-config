# Shadowrocket 新设备配置教程

适用于 iPhone、iPad 和 Mac，使用 Shadowrocket 正式版 **2.2.92 或更新版本**。本教程导入仓库中的完整配置，不需要逐条添加规则，也不需要另行填写 DNS。

## 1. 准备节点并备份

1. 在「首页」添加自己的节点或订阅，并选择要使用的节点。配置文件负责 DNS 和分流，节点仍需自己添加。
2. 已有配置的设备，进入「配置」，点击旧配置的更多信息按钮，选择「复制」或「导出配置」备份。
3. 导入期间保持 Shadowrocket 未连接。准备连接时，先关闭设备上其他正在运行的代理 VPN，避免同时接管网络。

## 2. 下载并使用配置

先选择版本，两版路由规则相同，仅默认及备用 DNS 不同：

| 版本 | 文件 | 未知域名默认 DNS |
| --- | --- | --- |
| 通用版 | [shadowrocket.conf](../shadowrocket.conf) | 阿里 DoH，优先照顾国内 CDN 调度 |
| 激进版 | [shadowrocket-aggressive.conf](../shadowrocket-aggressive.conf) | Cloudflare DoH，经代理查询 |

下面三种导入方式任选一种，**推荐方式一**，方便以后在软件中更新配置。

### 方式一：在 Shadowrocket 中从链接下载（推荐）

1. 打开 Shadowrocket →「配置」→ 右上角添加按钮。
2. 按选择的版本粘贴对应完整地址，选择「下载」：

   通用版：<https://raw.githubusercontent.com/zenvor/shadowrocket-config/main/shadowrocket.conf>

   激进版：<https://raw.githubusercontent.com/zenvor/shadowrocket-config/main/shadowrocket-aggressive.conf>

   新版本需在仓库发布后才能从这些链接下载；本地修改不会自动更新线上文件。

3. 下载完成后，按下方「导入后确认」选择并使用配置。

粘贴的是 raw 文件链接，不是 GitHub 的 `blob/main` 网页链接。不要在首页把它当作节点订阅添加。

### 方式二：下载文件后用 Shadowrocket 打开

1. 在浏览器打开上面的链接，将所选版本下载保存为对应的 `.conf` 文件。
2. 在下载列表或文件管理器中选择该文件，将打开方式选择为 Shadowrocket，软件会自动导入；手机也可以通过分享菜单选择 Shadowrocket。
3. 导入完成后，按下方「导入后确认」选择并使用配置。

### 方式三：在 Shadowrocket 中手动导入文件

1. 在浏览器打开上面的链接，将所选版本下载保存为对应的 `.conf` 文件。
2. 打开 Shadowrocket →「配置」→「导入」，手动选择下载好的 `.conf` 文件。Mac 的入口显示为「导入…」；部分手机版本可能显示为「从云导入」。
3. 导入完成后，按下方「导入后确认」选择并使用配置。

### 导入后确认

1. 在「配置」页面点击导入的 `shadowrocket.conf` 或 `shadowrocket-aggressive.conf` →「使用配置」。
2. 确认该文件出现默认圆点和使用中的勾选标记。
3. 返回「首页」，确认「全局路由」选中的是 **「配置」**，并确认选中了自己的节点。软件初始化时通常已默认选择「配置」。

菜单文字可能随系统语言和版本变化。

## 3. DNS 已经包含在配置中

| 用途 | 通用版 | 激进版 |
| --- | --- | --- |
| 默认及备用 DNS | 阿里 DoH | Cloudflare DoH，经当前代理 |
| 已匹配直连域名的 DNS | 阿里 DoH | 阿里 DoH |
| 代理节点自身的域名解析 | 阿里 DoH | 阿里 DoH |

两版的共同设置，无需手动添加：

```ini
direct-dns-server = https://dns.alidns.com/dns-query
proxy-dns-server = https://dns.alidns.com/dns-query
dns-direct-system = false
dns-direct-fallback-proxy = false
```

通用版的默认及备用设置：

```ini
dns-server = https://dns.alidns.com/dns-query
fallback-dns-server = https://dns.alidns.com/dns-query
```

激进版的默认及备用设置：

```ini
dns-server = https://cloudflare-dns.com/dns-query#proxy
fallback-dns-server = https://cloudflare-dns.com/dns-query#proxy
```

每项只填写一个 DNS 地址，避免同一项多个服务器产生并行查询；A/AAAA 查询、重试或备用查询仍可能产生多条日志。备用项与本版默认项使用同一个地址，不提供独立服务器冗余；保留该项是为了避免清空后回退到系统 DNS。激进版的国内专用 DNS 失败也可能回退到 Cloudflare。

未知域名触发 IP 分流时，通用版使用阿里，激进版使用经代理的 Cloudflare；命中代理域名规则的请求可能直接交给节点远端解析，所以并非每个 AI 请求都会出现在本地 DoH 查询日志里。

## 4. 当前分流行为

规则按顺序匹配，前面的规则优先：

| 情况 | 行为 |
| --- | --- |
| 命中 OpenAI / ChatGPT、Anthropic / Claude 或共享依赖前置规则 | 代理，即使目标是中国 IP |
| 命中默认配置中的其他特殊规则 | 使用该规则指定的代理或直连策略 |
| 未被前置规则覆盖的 `.cn` 域名 | 直连 |
| 命中国内域名直连规则 | 直连，使用直连域名专用 DNS |
| 未命中前面规则，只有域名、需要判断 IP | 解析真实 IP；命中中国 IP 规则则直连，否则最终代理 |
| 未命中前面规则，已有真实 IP | 命中中国 IP 规则则直连，否则最终代理 |

原版 Apple IP、Apple 服务和局域网等特殊直连规则保留，不是所有未知请求都只看中国 IP 归属。配置还保留系统隧道排除路由，部分地址可能不进入规则匹配。

两版都不是全局代理。通用版的未知域名可能受国内 DNS 答案影响；激进版可能将国内 App 的未知域名调度到海外 CDN，再按 IP 走代理。

核心域名的后缀规则覆盖其子域名，但未来出现的新独立 AI 域名仍需补充，未覆盖域名遵循后续分流规则。共享依赖规则也会影响其他应用访问这些服务。

HTTP3/QUIC 拦截规则保持原版的注释状态，**当前未启用**。不要为了导入本配置额外开启它。

## 5. 连接后验证

准备好后在首页开启连接；首次连接按系统提示添加 VPN 配置。检查：

- ChatGPT / Claude 的登录、对话、文件上传及下载。
- 国内视频播放、切换，以及购物和图片上传。
- Wi-Fi 与移动网络切换；关闭 VPN 后常用应用是否正常联网。

出现异常时，可在「数据」中查看代理日志和 DNS 日志；需要更多信息时，在「设置」→「诊断」开启日志记录，复现后导出 VPN 日志。对照发生时间、域名、DNS 答案、命中规则和出站节点判断。节点延迟数字或页面能打开，不能单独证明全部分流正确。

此前双 DNS 配置的实测日志已确认未知域名经国外 DNS 解析后，中国 IP 直连、其余代理；**本次两版单 DNS 配置尚未完成新的设备导入和联网验收**。

## 6. 更新与回退

从 URL 添加的配置，可点击文件选择「更新配置」，完成后重新选择「使用配置」并检查标记。更新会覆盖对该文件所做的本地修改，先备份自己的调整。仅点击「使用配置」不等于重新下载仓库中的完整文件。

从本地文件导入的配置，更新时重新下载上面的 raw 文件并导入，再选择新配置使用。仓库有更新不代表设备已经同步。

切换版本时，在「配置」选择对应文件 →「使用配置」，确认默认圆点和勾选标记，再重新连接并重启受影响的应用。

需要回退时，在「配置」中选择备份文件 →「使用配置」，然后重连并重启受影响的应用。

## 核对依据

- [通用版完整配置](../shadowrocket.conf)
- [激进版完整配置](../shadowrocket-aggressive.conf)
- [Shadowrocket 官方发布频道：直连域名专用 DNS](https://t.me/s/ShadowrocketNews?before=1620)
- [Shadowrocket 官方发布频道：节点域名 DNS](https://t.me/s/shadowrocketnews?after=931)
- [LOWERTOP 社区手册](https://github.com/LOWERTOP/Shadowrocket/wiki)：非官方资料，用于核对下载、更新入口及 DNS 行为说明；本次 Mac 本地导入流程另已通过界面核对，手机流程仍待实际验证。
- [OpenAI 网络要求](https://help.openai.com/zh-hans-cn/articles/9247338-network-recommendations-for-chatgpt-errors-on-web-and-apps)
- [Claude Code 网络要求](https://code.claude.com/docs/en/network-config#network-access-requirements)
- [Claude Desktop 网络要求](https://code.claude.com/docs/en/desktop#network-access-requirements)
