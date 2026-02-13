# 参考项目与规则设计对比

本文档以 **SimpRead（简悦）** 为例，简述其规则存储、结构、匹配与管理方式，并与 Pure Read / pure-read-rules 对比，供贡献者与主项目实现参考。

## SimpRead 规则体系概览

- **存储**：单文件 `website_list.json`，内含 `sites` 数组；同时从远程 `http://sr.ksria.cn/website_list_v4.json` 拉取并合并（GetRemote），本地与远程合并后写入 chrome.storage。
- **结构**：每条 site 含 `name`、`url`（minimatch 模式，如 `http*://*.cnbeta.com/articles/*/*`）、`title`、`desc`、`include`、`exclude`；可选 `avatar`、`paging`（论坛/分页）。选择器可为纯 CSS 或 `[[{$('selector')}]]` 在页面内执行 jQuery 取内容。
- **匹配**：PureRead 使用 **minimatch** 匹配当前页完整 URL 与 `site.url`；sites 分 global / person / custom / local。
- **管理**：独立「站点管理器」页（sitemgr）：新建/编辑/删除站点、从远程同步列表；支持从当前页「保存/提交」为站点（save_site、pending_site、temp_site 消息到选项页编辑）。

## 与 Pure Read / pure-read-rules 对比

| 维度 | SimpRead | Pure Read / pure-read-rules |
|------|----------|-------------------------------------|
| 规则存储 | 单文件 + 远程整份合并 | index.json + 按 host 分文件，按需拉取单条 |
| URL 匹配 | minimatch 完整 URL | host 短形式，`hostname.includes(host)` |
| 选择器 | CSS + `[[{$()}}]]` 页面内执行 | 仅 CSS 选择器 |
| 分类 | 无显式分类 | novel / news / tech / general |
| 管理 | 站点管理器页 + 从当前页提交 | 规则市场安装/卸载 + 规则管理 Tab（主项目） |
| 规则来源 | 本地 + 远程合并 | 内置 + 市场 + 自动 + 用户，四层优先级 |

## 可借鉴点（供主项目实现参考）

- **从当前页生成规则**：SimpRead 的 save_site / temp_site 流程可参考，在主项目中实现「从当前页生成规则草稿」：content 采集 host 与候选选择器，生成符合 [rule-schema](rule-schema.md) 的 JSON，可写入 RuleSystem 或导出供用户提交。
- **规则管理 UI**：SimpRead 的 sitemgr 提供独立页的站点列表与表单编辑；主项目可在规则市场/选项页中增加规则管理 Tab，列表展示、编辑 host 与各选择器、删除、导入导出。
- **版本与更新**：本仓库通过 index 的 version/updatedAt 与扩展侧 24h 缓存控制更新；SimpRead 通过远程整份合并更新。两种方式各有取舍，主项目保持「按需拉取 + 缓存」即可。

## 参考 website_list_v4.json 完善本仓库规则

主项目中的 `simpread/website_list_v4.json` 可作为规则来源参考。本仓库优先采用其中**仅使用纯 CSS 选择器**的条目进行转换，含 `[[{$()}}]]` 等页面内脚本的条目不转换。URL 匹配方式与 pathPattern、host 约定与 SimpRead 不同，详见 [simpread-conversion.md](simpread-conversion.md)。
