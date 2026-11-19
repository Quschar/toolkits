# 修复 home.lenovo.com 访问 — 增加公司 DNS 策略并保留可回滚备份

**Branch:** `N/A`
**Repo:** `local Clash Verge configuration`
**Date:** 2026-11-19
**Author:** wangzy136 (Zhiyuan ZY Wang)
**Status:** 已验证
**Keywords:** Clash Verge, mihomo, home.lenovo.com, DNS policy, nameserver-policy, 10.10.8.8, 10.122.248.227, rollback, backup

---

## 1. Symptom / 目标
`home.lenovo.com` 已命中 `DOMAIN-SUFFIX,lenovo.com,DIRECT`，但 Clash 内置公共 DNS 返回 NXDOMAIN，导致 mihomo 无法建立直连。

## 2. 排查过程 / 方案
保留原始配置后，在当前订阅使用的持久化 Merge Profile 中增加精确 DNS 策略：

```yaml
dns:
  nameserver-policy:
    'home.lenovo.com':
      - 10.10.8.8
```

只为 `home.lenovo.com` 指定公司 DNS，不把全部 `lenovo.com` 交给内部 DNS，降低影响范围。

## 3. 实现
修改文件：

| 文件 | 作用 |
|---|---|
| `~/.local/share/io.github.clash-verge-rev.clash-verge-rev/profiles/mTfz5criP99e.yaml` | 持久化 Merge Profile，订阅重载时保留 DNS 策略 |
| `~/.local/share/io.github.clash-verge-rev.clash-verge-rev/clash-verge.yaml` | 当前运行中的 mihomo 配置，已同步策略并重载 |

回滚备份目录：

```text
/home/wangzy136/.local/share/io.github.clash-verge-rev.clash-verge-rev/clash-verge-rev-backup/2026-11-19-144219-home-lenovo-dns/
```

备份内容：

- `mTfz5criP99e.yaml.before`
- `clash-verge.yaml.before`
- `rem8QudmPYy4.yaml.before`

## 4. 验证
mihomo 配置重载接口返回 HTTP `204`。

DNS 现在返回公司内部地址：

```text
home.lenovo.com → myhub.gtm.lendns.com → 10.122.248.227
```

经 Clash mixed port 访问成功：

```text
clash: http=302 ssl=0
```

原有直连规则仍为第 0 条：

```text
DomainSuffix lenovo.com DIRECT
```

## 5. 回滚
恢复持久化 Merge Profile，并用备份的运行配置重载：

```bash
base="$HOME/.local/share/io.github.clash-verge-rev.clash-verge-rev"
backup="$base/clash-verge-rev-backup/2026-11-19-144219-home-lenovo-dns"
cp "$backup/mTfz5criP99e.yaml.before" "$base/profiles/mTfz5criP99e.yaml"
cp "$backup/clash-verge.yaml.before" "$base/clash-verge.yaml"
curl --unix-socket /tmp/verge/verge-mihomo.sock \
  -X PUT http://localhost/configs \
  -H 'Content-Type: application/json' \
  --data "{\"path\":\"$base/clash-verge.yaml\",\"force\":true}"
```

该备份包含修复前的 Merge Profile、运行配置和已有 Lenovo 直连规则文件。
