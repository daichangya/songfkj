# 分类说明

规则按站点类型分为四类，用于规则市场展示与目录组织。

## 分类列表

| id | name | 说明 |
|----|------|------|
| novel | 小说阅读 | 连载、章节型站点，通常需要下一章/上一章/目录 |
| news | 新闻资讯 | 门户、资讯站，单篇正文为主 |
| tech | 技术博客 | 博客、社区、技术文章，单篇正文 + 作者 |
| general | 通用站点 | Medium、WordPress、HN、GitHub 等通用模板 |

## 何时用哪个

- **novel**：章节页有明显「下一章」「上一章」「目录」的站点；规则中应尽量提供 `nextSelector`、`prevSelector`、`tocSelector`。
- **news**：新闻门户文章页，以正文 + 标题 + 作者为主，一般不提供章节导航。
- **tech**：技术博客、问答、专栏等，单篇文章；通常有 `authorSelector`，一般不填 next/prev/toc。
- **general**：通用内容站（如 Medium、Substack、WordPress 默认主题、Hacker News、GitHub README/Issues），按需提供作者与排除选择器。

## 与章节导航的关系

- Pure Read 的「下一章 / 上一章 / 目录」**仅当规则中包含** `nextSelector`、`prevSelector`、`tocSelector` 时才会显示。
- **tech** 类规则通常不包含这些选择器，避免在单篇文章页误显示章节导航。
- **novel** 类应尽量提供，以支持连续阅读与目录跳转。
