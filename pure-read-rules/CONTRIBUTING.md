# 贡献指南

欢迎为 pure-read-rules 贡献新规则或改进现有规则。

## 如何贡献

1. **Fork** 本仓库
2. 新建分支（如 `feature/add-xxx-rule`）
3. 新增或修改规则文件，并更新 `index.json`
4. 提交 **Pull Request**

## 新增一条规则

### 1. 新建规则文件

在 `rules/{category}/{host}.json` 新建文件：

- `category` 为 `novel`、`news`、`tech`、`general` 之一，见 [docs/categories.md](docs/categories.md)
- `host` 与索引中一致，使用**短形式、无 TLD**；若 host 含点（如 `dev.to`），文件名用 `-` 替代，如 `dev-to.json`、`zhuanlan-zhihu.json`

### 2. 编写规则内容

文件结构为顶层 `meta` + `rule`：

- **meta**：`name`、`description`、`author`、`version`、`category`、`updatedAt`
- **rule**：`host`、`titleSelector`、`contentSelector`，以及可选的 `authorSelector`、`excludeSelectors`、`nextSelector`、`prevSelector`、`tocSelector`

格式与字段说明见 [docs/rule-schema.md](docs/rule-schema.md)。  
host 命名约定见 [docs/host-convention.md](docs/host-convention.md)。

### 3. 更新 index.json

在 `index.json` 的 `rules` 数组中增加一条：

- `host`、`name`、`category`、`description`、`author`、`version`、`updatedAt`、`downloads`（可先填 0）、`selectors`（该规则实际使用的选择器键名列表，如 `["title","content","next","prev","toc","exclude"]`）

并更新对应 `categories` 中该分类的 `count`。

## host 约定

- 使用**短形式、不含 .com/.cn/.net**（如 `qidian`、`sina`、`csdn`）
- 子站保留子域（如 `zhuanlan.zhihu`、`news.ycombinator`）
- 品牌含点保留（如 `dev.to`）

详见 [docs/host-convention.md](docs/host-convention.md)。

## 测试建议

1. 在 Pure Read 中加载本仓库（本地打包或设置远程规则市场地址）
2. 在目标站点打开任意文章/章节页，进入阅读模式
3. 确认正文、标题、作者（若有）提取正确；若为小说站，确认「下一章 / 上一章 / 目录」链接正确

如有问题，可在 PR 中说明测试的 URL 与现象。
