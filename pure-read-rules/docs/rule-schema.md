# 规则 JSON 结构说明

本文档描述规则市场索引 `index.json` 与单条规则文件的结构、字段含义及示例。

## index.json 结构

| 字段 | 类型 | 说明 |
|------|------|------|
| version | string | 索引版本号，如 `"1.0.0"` |
| updatedAt | string | 索引更新时间，ISO 8601，如 `"2026-02-11T12:00:00Z"` |
| categories | array | 分类列表，每项含 id、name、icon、count |
| rules | array | 规则摘要列表，用于市场展示与按 host 拉取详情 |

### categories 每项

| 字段 | 类型 | 说明 |
|------|------|------|
| id | string | 分类 id：novel / news / tech / general |
| name | string | 显示名称 |
| icon | string | 图标（如 emoji） |
| count | number | 该分类下规则数量 |

### rules 每项（摘要）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| host | string | 是 | 站点 host 短形式，与规则文件对应 |
| name | string | 是 | 规则显示名称 |
| category | string | 是 | 分类 id |
| author | string | 否 | 作者 |
| description | string | 否 | 规则说明 |
| version | string | 否 | 规则版本 |
| updatedAt | string | 否 | 规则更新时间 |
| downloads | number | 否 | 下载量（可填 0） |
| selectors | string[] | 否 | 该规则使用的选择器键名，如 `["title","content","next","prev","toc","exclude"]` |

扩展拉取单条规则时，会请求：`{baseUrl}rules/{category}/{filename}.json`，其中 `filename = host` 中的 `.` 替换为 `-`（如 `dev.to` → `dev-to.json`）。

---

## 单条规则 JSON 结构

每个规则文件为单一 JSON 对象，包含顶层 **meta** 与 **rule**。

### meta（元信息）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| name | string | 是 | 规则名称 |
| description | string | 否 | 规则描述 |
| author | string | 否 | 作者 |
| version | string | 否 | 版本号 |
| category | string | 是 | 分类 id |
| updatedAt | string | 否 | 更新时间 |

### rule（选择器规则）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| host | string | 是 | 与 meta.category 及文件名一致，短形式 |
| pathPattern | string | 否 | 路径前缀，如 `"/article"`、`"/posts"`；不填表示匹配该 host 下所有路径（一站点多规则时用于区分列表页/详情页等） |
| name | string | 否 | 规则展示名，如「列表页」「详情页」，便于同 host 多条规则区分 |
| titleSelector | string | 是 | 标题 CSS 选择器，多个用逗号分隔 |
| contentSelector | string | 是 | 正文 CSS 选择器，多个用逗号分隔 |
| authorSelector | string | 否 | 作者选择器 |
| excludeSelectors | string[] | 否 | 从正文中排除的节点选择器 |
| nextSelector | string | 否 | 下一章链接选择器 |
| prevSelector | string | 否 | 上一章链接选择器 |
| tocSelector | string | 否 | 目录/章节目录链接选择器 |

- 仅当存在 `nextSelector` / `prevSelector` / `tocSelector` 时，扩展会显示章节导航；技术博客等通常不填，避免误显示。
- 选择器可写多个，用英文逗号分隔，扩展会按顺序尝试直至匹配。
- **一站点多规则**：同一 host 可在单文件中提供多条规则（见下文「多规则单文件」），通过 `pathPattern` 做路径前缀匹配；无 `pathPattern` 的规则作为该 host 的默认规则（匹配任意路径）。

---

## 示例一：小说站（novel）

文件：`rules/novel/biquge.json`

```json
{
  "meta": {
    "name": "笔趣阁",
    "description": "笔趣阁系列（biquge/xbiquge等）正文提取规则",
    "author": "Pure Read Team",
    "version": "1.0.0",
    "category": "novel",
    "updatedAt": "2026-02-01"
  },
  "rule": {
    "host": "biquge",
    "titleSelector": ".bookname h1, .content_read h1",
    "contentSelector": "#content, #chaptercontent, .chapter_content",
    "excludeSelectors": [".readinline", ".bottem", "script", "style"],
    "nextSelector": "#A3, .bottem1 a:last-child, .next_url",
    "prevSelector": "#A1, .bottem1 a:first-child, .prev_url",
    "tocSelector": "#A2, .bottem1 a:nth-child(2)"
  }
}
```

---

## 示例二：技术博客（tech）

文件：`rules/tech/csdn.json`

```json
{
  "meta": {
    "name": "CSDN",
    "description": "CSDN博客正文提取规则",
    "author": "Pure Read Team",
    "version": "1.0.0",
    "category": "tech",
    "updatedAt": "2026-02-01"
  },
  "rule": {
    "host": "csdn",
    "titleSelector": "#articleContentId, h1.title-article",
    "contentSelector": "#content_views, #article_content, .article_content",
    "authorSelector": ".follow-nickName, .user-info a",
    "excludeSelectors": ["script", "style", ".csdn-side-toolbar", ".recommend-box", ".blog-tags-box", ".template-box", ".comment-box"]
  }
}
```

技术博客通常不包含 next/prev/toc，只做正文与作者提取。

---

## 多规则单文件（一站点多规则）

同一站点（同一 host）可在一个 JSON 文件中提供多条规则，例如列表页一条、详情页一条。扩展拉取该文件后会安装全部规则。

### 格式约定

- **单条规则**：沿用 `{ "meta": {...}, "rule": {...} }`，安装一条。
- **多条规则**：使用 `{ "meta": {...}, "rules": [ rule1, rule2, ... ] }`；若同时存在 `rule` 与 `rules`，以 **rules** 为准（安装 rules 数组中的全部）。
- 每条 rule 可带 `pathPattern`、`name`；扩展匹配时用当前页 `pathname` 做前缀匹配，更具体的 pathPattern 优先。

### 示例：同一 host 下列表页与详情页

文件：`rules/general/example.json`

```json
{
  "meta": {
    "name": "Example 站",
    "description": "列表页与详情页分别配置",
    "category": "general"
  },
  "rules": [
    {
      "host": "example",
      "pathPattern": "/article",
      "name": "详情页",
      "titleSelector": "h1.article-title",
      "contentSelector": ".article-body",
      "authorSelector": ".author-name"
    },
    {
      "host": "example",
      "pathPattern": "/list",
      "name": "列表页",
      "titleSelector": "h2.item-title",
      "contentSelector": ".item-summary"
    }
  ]
}
```

列表页规则适用于路径以 `/list` 开头的页面，详情页规则适用于以 `/article` 开头的页面；其他路径若有「无 pathPattern」的默认规则则使用默认规则。