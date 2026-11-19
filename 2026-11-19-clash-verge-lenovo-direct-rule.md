# Clash Verge 增加 Lenovo 直连规则 — lenovo.com 及其子域名直连

**Branch:** `N/A`
**Repo:** `local Clash Verge configuration`
**Date:** 2026-11-19
**Author:** wangzy136 (Zhiyuan ZY Wang)
**Status:** 已验证
**Keywords:** Clash Verge, mihomo, lenovo.com, DOMAIN-SUFFIX, DIRECT, proxy rule, local config

---

## 1. 目标
让所有以 `lenovo.com` 为后缀的域名绕过代理、直接连接。

## 2. 排查过程 / 方案
- 运行中的核心为 `verge-mihomo`，配置目录为 `/home/wangzy136/.local/share/io.github.clash-verge-rev.clash-verge-rev`。
- 当前远程订阅通过规则增强文件 `profiles/rem8QudmPYy4.yaml` 加载自定义规则。
- 使用 `prepend` 而不是追加到规则尾部，确保规则先于广告拒绝规则及最终 `MATCH,Proxy` 生效。

## 3. 实现
涉及文件：

| 文件 | 作用 |
|---|---|
| `~/.local/share/io.github.clash-verge-rev.clash-verge-rev/profiles/rem8QudmPYy4.yaml` | 持久化的 Clash Verge 规则增强配置 |
| `~/.local/share/io.github.clash-verge-rev.clash-verge-rev/clash-verge.yaml` | 当前生成并运行中的 mihomo 配置 |

新增规则：

```yaml
prepend:
  - DOMAIN-SUFFIX,lenovo.com,DIRECT
```

同时将当前运行配置的 `rules` 首条设置为：

```yaml
- DOMAIN-SUFFIX,lenovo.com,DIRECT
```

## 4. 验证
通过 `/tmp/verge/verge-mihomo.sock` 调用 mihomo 配置重载接口，返回 HTTP `204`。随后查询 `/rules`，确认运行中的第 0 条规则为：

```text
DomainSuffix  lenovo.com  DIRECT
```

## 5. 遗留
无。规则覆盖 `lenovo.com` 及其子域名；规则变更已加载到当前运行中的 mihomo。
