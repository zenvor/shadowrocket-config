# shadowrocket-config

Shadowrocket（iOS/macOS）完整分流配置，基于 2.2.92 导出的默认配置，补充 DNS 与 OpenAI / ChatGPT、Anthropic / Claude 前置代理规则。

新设备配置请看：[Shadowrocket 配置教程](docs/shadowrocket-setup.md)。

## 版本要求

使用 Shadowrocket 正式版 **2.2.92 或更新版本**（iOS/macOS）。直连域名专用 DNS 功能从 2.2.91 测试版 build 3410 加入，本配置以包含该功能的正式版 2.2.92 为最低要求。

## 文件及使用

1. 先复制备份现用配置。
2. [shadowrocket.conf](shadowrocket.conf) 是可直接导入的完整配置，包含 662 条规则。导入后选择“使用配置”，全局路由选择“配置”，代理使用自己已添加的节点。
3. 默认及备用 DNS 使用经代理的 Google / Cloudflare DoH；直连域名与节点域名使用阿里 / 腾讯 DoH。保留默认的国内域名规则、.cn 直连、中国 IP 分流及 FINAL,PROXY；HTTP3/QUIC 拦截保持默认注释状态。

`PROXY` 使用应用所选代理；不要导入社区帖子的 `CLAUDE0409 = direct` 或 `FINAL,DIRECT`。

## DNS 差异

- `direct-dns-server` 供已命中 DIRECT 域名规则的查询，IP/GEOIP 触发的查询仍使用 `dns-server`。
- `proxy-dns-server` 解析代理节点地址，不控制代理目标网站解析。
- `#proxy` 是 Shadowrocket 的 DNS 代理标记，不能替换成 Mihomo 的策略组语法。
- 命中 PROXY 的域名可能交给远端节点解析，因此不保证所有 AI 域名都经本地指定的 DoH。
- fallback-dns-server 是共同备用，国内专用 DNS 失败也可能用国外备用，与 Mihomo 的失败路径不完全相同。
- 保留 .local/.lan 等实际内网名称需要的局域网 DNS 例外，公共国内 DoH 不一定能解析。

精确参数语法依据 LOWERTOP **社区手册，非官方手册**；App Store 官方确认相关功能，但未发布同等完整配置语法。未使用未经实机验证的隐藏参数。

## 覆盖与维护

核心后缀自动覆盖新子域，未来新独立域名需补充；仅有 IP 无域名的请求仍按后续 IP 分流。共享主机和 datadog/sift 关键词会影响其他应用请求。本配置不是账号安全保证。

本仓库已公开，可匿名访问 GitHub 和 raw 文件。完整配置已在 macOS Shadowrocket 2.2.92 导入并设为默认，尚未进行联网测试或手机验收；导入成功不等于实际出站行为已验证。

来源：
- https://help.openai.com/zh-hans-cn/articles/9247338-network-recommendations-for-chatgpt-errors-on-web-and-apps
- https://code.claude.com/docs/en/desktop#network-access-requirements
- https://code.claude.com/docs/en/network-config#network-access-requirements
- https://x.com/wlzh/status/2108017417670860900
- https://github.com/LOWERTOP/Shadowrocket/wiki
- https://apps.apple.com/us/app/shadowrocket/id932747118
- https://shadowlaunch.com/

- https://t.me/s/ShadowrocketNews?before=1620
