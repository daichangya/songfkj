# SimpRead 规则参考与转换说明

本文档说明如何参考 SimpRead 的 `website_list_v4.json`（见主项目 `simpread/website_list_v4.json`）完善本仓库规则。仅**纯 CSS 选择器**的条目可自动/半自动转换为 Pure Read 规则；含 `[[{$()}}]]` 等页面内脚本的条目不转换。

## 字段映射

| SimpRead 字段 | Pure Read 字段 | 说明 |
|---------------|-------------------|------|
| `name` | `host` | 取短形式：去掉 `.com`/`.cn` 等 TLD；子域保留如 `news.36kr`；与 [host-convention.md](host-convention.md) 一致 |
| `url` | `pathPattern`（可选） | 从 minimatch 推路径前缀，如 `http*://*.cnbeta.com/articles/*/*` → `/articles/`；无法可靠推导时可省略 |
| `title` | `titleSelector` | 仅当为纯 CSS 时转换：`<header.title h1>` → `header.title h1`；含 `[[{$` 或 `[[/` 的条目不转换 |
| `include` | `contentSelector` | 同上，纯 CSS 如 `<div class='detail'>` → `div.detail` |
| `exclude` | `excludeSelectors` | 数组；仅取纯 CSS 字符串项，过滤掉 `[[...]]`、正则、`[['...']]` 等 |
| `desc` | — | 当前 schema 无摘要字段；不映射或仅作规则 description 文案参考 |

## 不可转换情形

- `title` / `include` / `exclude` 任一项含 `[[{$`、`[[/`、`[[\`` 的站点：仅作参考，不生成规则文件（或仅取其中纯 CSS 的 exclude 项）。
- SimpRead 使用 minimatch 全 URL 匹配，本仓库使用 `hostname.includes(host)` + 可选 `pathPattern` 路径前缀，同一主域下多子路径可拆成多条规则（不同 pathPattern）或合并为一条无 pathPattern 的默认规则。

## SimpRead 选择器写法转 CSS

- SimpRead 常用 `<'tag class='classname'>` 形式，转换为标准 CSS：`tag.classname`（注意 class 中的空格表示多 class，如 `class='a b'` → `.a.b`）。
- 带 id：`<div id='xxx'>` → `div#xxx`。
- 多个选择器用英文逗号分隔，与本仓库一致。

## 示例：cnbeta.com

**SimpRead 片段：**

```json
"name": "cnbeta.com",
"url": "http*://*.cnbeta.com/articles/*/*",
"title": "[[{$('header.title h1').text()}]]",
"include": "<div class='cnbeta-article-body'>",
"exclude": ["<div class='clearfix'>", "<div class='rating_box'>", ...]
```

**转换后（Pure Read）：**

- `host`: `cnbeta`
- `pathPattern`: `/articles/`
- `titleSelector`: `header.title h1`（从 SimpRead 的 jQuery 选择器提取）
- `contentSelector`: `div.cnbeta-article-body`
- `excludeSelectors`: 仅保留纯 CSS 项，如 `div.clearfix`, `div.rating_box`, `div.article-relation`, `div.topic`, `div.tac`, `div.article-share-code`, `div.article-global`, `div.article-topic`

## 参考

- [rule-schema.md](rule-schema.md) — 本仓库规则结构
- [host-convention.md](host-convention.md) — host 与 pathPattern 约定
- [reference-projects.md](reference-projects.md) — 与 SimpRead 等对比概览
