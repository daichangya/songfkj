# host 命名约定

规则中的 `host` 及规则文件名需遵循以下约定，以便与扩展匹配逻辑一致。

## 约定

- **短形式、不含 TLD**：不写 `.com`、`.cn`、`.net` 等，例如：
  - ✅ `qidian`、`sina`、`csdn`
  - ❌ `qidian.com`、`www.sina.com.cn`
- **子站保留子域**：若站点按子域区分，保留子域部分，例如：
  - ✅ `zhuanlan.zhihu`（知乎专栏）、`news.ycombinator`（Hacker News）
- **品牌含点保留**：品牌名本身含点时保留，例如：
  - ✅ `dev.to`

## 匹配方式

扩展内使用 **`hostname.includes(host)`** 匹配当前页与规则：

- 因此 `host` 写短形式即可匹配 `www.xxx.com`、`m.xxx.com`、`xxx.com` 等。
- 子域站点用 `zhuanlan.zhihu` 可匹配 `zhuanlan.zhihu.com`。

## 文件名

- 规则文件路径：`rules/{category}/{filename}.json`
- **filename**：将 `host` 中的 `.` 替换为 `-`，`/` 替换为 `-`，与扩展拉取时的候选路径一致。例如：
  - `dev.to` → `dev-to.json`
  - `zhuanlan.zhihu` → `zhuanlan-zhihu.json`
  - `news.ycombinator` → `news-ycombinator.json`

扩展 background 中拉取规则详情时使用：  
`filename = host.replace(/\./g, '-').replace(/\//g, '-')`，  
因此 **index.json 中的 host** 与 **规则文件中的 rule.host** 必须一致，文件名 = 该 host 做上述替换后的值。

---

## URL 匹配方式说明

本仓库与扩展采用 **host 短形式 + `hostname.includes(host)`** 的匹配方式，与采用 **minimatch** 等完整 URL 模式匹配的扩展（如 SimpRead）不同：

- **本方案**：只关心「当前页 hostname 是否包含该 host」，不区分路径（如 `/articles/123` 与 `/news/456` 同属一个 host）。优点是实现简单、无需引入 minimatch、规则易写易读；若站点下不同路径需要不同规则，可拆成多条 host（如子域）或由主项目未来扩展 path 约定。
- **minimatch 方案**：可精确到 `http*://*.cnbeta.com/articles/*/*`，路径变化也可区分。适合站点结构复杂、同一主域下多套模板的场景；代价是规则书写与依赖更重。
- **取舍**：当前以「host 为主、简洁可维护」为先；路径级区分已通过可选 **pathPattern** 支持（见下）。

## 一站点多规则

同一 host 可配置多条规则，用 **pathPattern**（路径前缀）区分，例如列表页一条、详情页一条：

- 规则中可选字段 **pathPattern**：如 `"/article"`、`"/posts"`；当前页 `pathname` 以该前缀开头时命中该规则。
- 不填 pathPattern 的规则表示「该 host 下任意路径」的默认规则。
- 规则文件仍按 host 命名（如 `medium.json`）；单文件可为单条 `rule` 或多条 `rules` 数组，详见 [rule-schema](rule-schema.md)。
