# Pure Read 如何引用本仓库

Pure Read 主项目可通过「本地打包」或「远程规则市场」使用本仓库的规则。

## 本地打包

主项目将 pure-read-rules 以 **Git submodule** 或**复制**方式放入工作区（例如目录名为 `pure-read-rules` 或仍叫 `rules-repo`），构建时把该目录内容复制到扩展的 `dist/rules-repo/`，扩展从 `chrome.runtime.getURL('rules-repo/')` 加载。

- **Vite**：在 `vite.config.ts` 的 `viteStaticCopy` 中，将复制源设为 `pure-read-rules/**/*`（或 submodule 检出路径），目标为 `rules-repo`，这样打包后扩展内路径仍为 `rules-repo/`，无需改 background 代码。
- **Submodule 建议**：主项目执行 `git submodule add <pure-read-rules-url> pure-read-rules`，复制源指向 `pure-read-rules/**/*`；克隆主项目后需执行 `git submodule update --init` 以拉取规则，否则构建会失败。

## 远程规则市场（自建网站）

将本仓库发布到 GitHub 后，可将仓库内容部署到**自建网站**（任意支持静态文件的 Web 服务）。在 Pure Read 的 **background** 中设置远程基础 URL，扩展即可从自建网站拉取 `index.json` 与单条规则文件。

- **变量**：主项目 `src/background/index.ts` 中的 `MARKET_REMOTE_BASE_URL`。
- **留空**：使用本地打包的 `rules-repo/`（即 `chrome.runtime.getURL('rules-repo/')`）。
- **设为自建网站 base URL**：例如  
  `https://your-domain.com/pure-read-rules/`  
  扩展会请求：
  - `{baseUrl}index.json` 获取索引；
  - `{baseUrl}rules/{category}/{filename}.json` 获取单条规则（filename 为 host 中点替换为 `-`）。

只要自建网站按上述路径提供静态文件即可，无需打包规则进扩展即可使用本仓库最新规则。

## 版本与缓存

- **index.json** 的 `version`、`updatedAt` 可用于扩展侧缓存失效判断。
- 扩展已有规则市场缓存（主项目中 `MARKET_CACHE_TTL`，如 24 小时）；超过 TTL 会重新请求 index，再按需拉取规则详情。具体 TTL 以主项目 `src/background/index.ts` 配置为准。
