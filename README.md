# shadowrocket-config

专供 Shadowrocket（iOS/macOS）的域名规则与 DNS 配置片段，独立于 Mihomo 和 v2rayNG/v2rayN 仓库。不包含节点或凭据。

## 版本要求

使用 Shadowrocket 正式版 **2.2.92 或更新版本**（iOS/macOS）。直连域名专用 DNS 功能从 2.2.91 测试版 build 3410 加入，本配置以包含该功能的正式版 2.2.92 为最低要求。

## 文件及使用

1. 先复制备份现用配置。
2. `rules/ai-proxy.conf` 包含 52 条已知 OpenAI/Anthropic 及共享依赖域名前置 PROXY 规则。合并到已有 `[Rule]` 区段，放在广告、国内域名和 IP 直连规则之前；不要重复创建 `[Rule]`。保留原有 UDP443 拦截在最前。
3. 保留自己的国内域名名单、私网/家庭局域网规则及其他特殊规则。当前文件不附带国内域名数据集，不能当作完整国内白名单使用。
4. `examples/routing-tail.conf` 是末尾规则示例，.cn 后缀直连；GEOIP CN 不加 no-resolve，使未命中域名规则的请求解析真实 IP 后分流；其余 FINAL,PROXY。替换原 FINAL，而不是加在旧 FINAL 后面；需要的规则不能放在 FINAL 后面。
5. `examples/dns.conf` 合并到已有 `[General]` 区段，未知域名/IP 分流查询默认用代理 Google/Cloudflare DoH；直连域名专用国内 DoH；节点域名独立解析。保存后使用该配置，全局路由选择“配置”。本仓库文件均为合并片段，不是含节点和国内名单的完整配置。

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

本仓库已公开，可匿名访问 GitHub 和 raw 文件。片段需人工合并，导入/更新不等于已加载生效。检查 DNS 上游、真实 IP、命中规则及出站日志；尚未在手机验收，未切换 VPN。

来源：
- https://help.openai.com/zh-hans-cn/articles/9247338-network-recommendations-for-chatgpt-errors-on-web-and-apps
- https://code.claude.com/docs/en/desktop#network-access-requirements
- https://code.claude.com/docs/en/network-config#network-access-requirements
- https://x.com/wlzh/status/2108017417670860900
- https://github.com/LOWERTOP/Shadowrocket/wiki
- https://apps.apple.com/us/app/shadowrocket/id932747118
- https://shadowlaunch.com/

- https://t.me/s/ShadowrocketNews?before=1620
