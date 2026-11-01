# nivo-plugins-artifacts

**插件产物仓**：市场服务（`nivo-market-api`／`apps/market-api`）和宿主读的那份索引就在这个仓库。

## 目录约定

```
index.json                 # 完整索引（人工可读，市场服务对外就转它）
index.min.json             # 压缩版（给宿主/CI 快速取）
manifests/<id>/<version>/plugin.json   # 完整清单（权限/设置/插槽声明）
artifacts/…                # 不进 git，见下
```

**产物（`*.js`）不提交进 git**：它们走 **GitHub Releases**（tag `plugins-latest`），
索引里的 `entry.url` 直指 Release 资产。理由：单 JS 产物 1~3 MB，几十个插件就是几百 MB，
塞进 git 会让克隆和 diff 都变慢；Release 资产天然走 CDN 且可覆盖上传。

## 谁写这个仓库

`nivo-team/dashboard` 的 `pnpm plugin:build && pnpm market:index --publish <本仓工作区>`
（`scripts/market/build-index.mjs`）。它会：

1. 重算每个产物的 sha256/size（不抄清单）；
2. 写 `index.json` / `index.min.json` / `manifests/**`；
3. 打印 `gh release upload` 与 `git` 的后续命令（网络操作交给人或 CI）。

## 谁读这个仓库

- **市场服务**：`GET /api/plugins` 读 `index.json`（带 TTL 缓存 + 本地兜底）；
- **宿主**：安装时按 `entry.url` 下载、**自己重算 sha256** 再入 IDB。

## 校验与格式

- `schemaVersion` 破坏性变更时 +1；
- 每条插件的 `entry` 必须有 `url` / `sha256` / `size`；
- `id` 与宿主 `PluginMeta.id` 一致（内置插件是单段 `ai`，第三方是 `owner/name`）。
