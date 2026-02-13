# pure-read-rules

Pure Read 扩展的站点规则仓库，用于正文提取与章节导航。本仓库与主项目（Pure Read）解耦，可独立开源、版本管理与社区贡献。

## 规则是什么

每条规则对应一个站点（或一批同构站点），通过 **host** 与 **CSS 选择器** 指定：

- **正文**：`contentSelector`、`titleSelector`、`authorSelector`
- **排除噪音**：`excludeSelectors`
- **章节导航**（可选）：`nextSelector`、`prevSelector`、`tocSelector`

扩展在匹配到当前页 host 后，优先用规则中的选择器提取正文；若规则包含下一章/上一章/目录选择器，会显示章节导航栏。

## 谁会用

- **Pure Read** 扩展：内置打包或从「规则市场」远程加载本仓库
- 其他兼容本格式的阅读/抓取工具（见 [docs/rule-schema.md](docs/rule-schema.md)）

## 仓库结构

```
pure-read-rules/
├── index.json          # 规则市场索引（分类 + 规则列表）
├── rules/
│   ├── novel/          # 小说站
│   ├── news/            # 新闻站
│   ├── tech/            # 技术博客
│   └── general/         # 通用站点
└── docs/                # 说明文档
```

- **index.json**：与扩展规则市场对接，包含 `version`、`updatedAt`、`categories`、`rules` 数组。
- **promo.json**：可选，用于推广或测试等，扩展可按需加载。
- **rules/{category}/{host}.json**：单条规则，`host` 与索引一致；文件名中 `.` 用 `-` 替代（如 `dev-to.json`）。

详见 [docs/rule-schema.md](docs/rule-schema.md)。

## 快速开始

- **使用规则**：Pure Read 可通过本地打包或远程（自建网站）加载本仓库，见 [docs/integration.md](docs/integration.md)。
- **贡献规则**：Fork 本仓库 → 在 `rules/{category}/` 下新增或修改 JSON → 更新 `index.json` → 提交 PR。详见 [CONTRIBUTING.md](CONTRIBUTING.md) 与 [docs/host-convention.md](docs/host-convention.md)。

## 与主项目关系

本仓库为**独立开源项目**，与 Pure Read 主仓库分离。主项目可通过：

- **本地**：Git submodule 或构建时复制到 `rules-repo/`，打包进扩展。
- **远程**：将本仓库内容部署到自建网站后，在主项目 `src/background/index.ts` 中将 `MARKET_REMOTE_BASE_URL` 设为该站点的 base URL，扩展从自建网站拉取规则。

详见 [docs/integration.md](docs/integration.md)。

### 发布到 GitHub 后（通过自建网站使用）

将本仓库推送到 GitHub 后，可将仓库内容部署到**自建网站**（任意支持静态文件的 Web 服务）。Pure Read 通过设置 **MARKET_REMOTE_BASE_URL** 指向该站点的 base URL 使用规则。

- **示例 base URL**：`https://your-domain.com/pure-read-rules/`（以 `/` 结尾）。
- **扩展请求方式**：扩展会请求 `{baseUrl}index.json` 与 `{baseUrl}rules/{category}/{filename}.json`，自建网站需能按此路径提供对应静态文件。
- **配置位置**：主项目 `src/background/index.ts` 中的 `MARKET_REMOTE_BASE_URL`。细节见 [docs/integration.md](docs/integration.md)。

## 许可

[MIT](LICENSE)
