# AGENTS.md

Hugo 站点，主题通过 Go Module 引用 **OINK**。所有建站、配置、内容与部署操作必须遵循 OINK 官方中文文档：<https://oink.pgsty.com/zh/docs/>。本文件只记录容易踩坑、且从代码难以一眼看出的仓库特有事实；与文档冲突时以文档和可执行配置为准。

## 必备上下文

- 不需要 Node/npm。构建只依赖 Git、Go 1.27+、Hugo Extended。工具由 `mise.toml` 固定（go 1.27、hugo-extended-withdeploy），先 `mise install`。
- 主题 OINK **不复制进本仓库**，以 Go Module 固定在 `go.mod` / `go.sum`（`github.com/pgsty/oink v1.1.0`）。两者都要提交。
- `defaultContentLanguage: zh` + `disableLanguages: [en, fr]`：**仅中文站点**，服务于站点根。英文同为声明但禁用，语言菜单不出现。英文内容以无后缀基础 `.md` 存放，被同名 `.zh.md` 覆盖；若新增一个只有英文、没有 `.zh.md` 对页的文件，它会以英文渲染进中文站且无告警，务必成对维护。

## 命令

```bash
hugo server                      # 本地预览 http://localhost:1313/

# 发布闸门（CI 用的严格构建，任何 warning 都会失败）：
hugo --cleanDestinationDir --gc --minify --environment production \
  --printPathWarnings --panicOnWarning

hugo mod graph | grep github.com/pgsty/oink   # 查看实际解析到的主题版本
```

`--panicOnWarning` 是硬约束：未开 `LLMSFULL` 却把输出挂在非顶层栏目等都会直接构建失败，本地改动后务必跑一次严格构建。

## 配置与内容约定

- 唯一生效的站点配置是根目录 `hugo.yaml`；`examples/` 语言 profile 已删除，不要再引用。
- `hugo.yaml` 的 `outputs` 是**按类型整体替换**，不是追加。要加/去 `markdown`、`LLMS`、`print` 时必须把该类型原本的格式一起写全，漏写会静默丢输出。front matter 里的 `outputs` 同理。
- 站点只发布中文（`defaultContentLanguage: zh`）。每个页面只保留一份内容文件，默认语言用无后缀 `.md`。`en`/`fr` 仅为让 Hugo 识别 `.en.md`/`.fr.md` 后缀而声明并禁用，不要为它们新增内容；首页数据只有 `data/home/zh.yaml`。
- 顶部导航来自各栏目根 `_index` 页的 `menus.main`（`identifier` / `parent` / `weight`）；删掉某栏目根就会从导航移除，记得同时清掉指向它的首页卡片与按钮。
- 评论只在文章单页开启：栏目列表页（`content/blog/_index.zh.md`、`content/blog/post/_index.zh.md`）在 front matter 写 `comments: false` 覆盖站点配置；新增列表页时要记得加，否则列表页也会出现评论区。
- 改动内容后按官方「Starter 仓库导览」的安全顺序：身份 → 语言 → 首页 → 内容/导航 → 品牌 → 集成 → 严格构建 → 部署，每层一次提交。

## 必须保留（否则破坏功能）

- `go.mod` + `go.sum`（固定并校验 OINK 版本）。
- `hugo.yaml` 中三项 Goldmark 设置（原生 Steps/Cards/Fields 与图片属性依赖它们）。
- `outputs` 中已开启的 `markdown` / `LLMS` / `print`（Agent 输出、全文包、打印表面）。
- workflow 里的 `fetch-depth: 0`（`enableGitInfo: true` 需要完整 Git 历史）。
- CI 的 `GOWORK: off` 与 `HUGO_MODULE_WORKSPACE: off`：本地 workspace 覆盖不得混入 CI。

## 生成物与部署

- 不要提交 `public/`、`resources/`、`.hugo_cache/`、`.hugo_build.lock`，均为构建产物，已在 `.gitignore`。
- 本地开发若用 `HUGO_MODULE_REPLACEMENTS` 覆盖主题，绝不能提交，也不能当作发布证明。
- 部署用 `.github/workflows/github-pages.yaml`：**仅手动触发**（`workflow_dispatch`，不再随 `main` 推送运行），在 Actions 里点 Run workflow。前置条件是仓库 **Settings → Pages → Source 选 GitHub Actions**。它用 `actions/configure-pages` 计算 `baseURL`（已绑自定义域名 `leafz.net` 时即站点根），并保留 `fetch-depth: 0`。Cloudflare Pages/Workers 的部署尝试与相关文件已全部移除，不要再引入。
- 严格构建的 `HUGO_VERSION` 固定为 `0.165.0`（见 workflow），与 `hugo.yaml` 的 `min: 0.160.1`；升级主题或 Hugo 时参考官方「版本升级」页。

## 站点现状与入口

已定制为「一叶方舟」个人站：身份 / URL / 全部集成在 `hugo.yaml`；首页数据 `data/home/zh.yaml`（`sections` 为 `hero` + 一个 `type: markdown, key: about` 的「关于本站」）；内容只剩 `content/blog/`（文章放 `content/blog/post/`）。站点 SCSS 覆盖在 `assets/scss/_styles_project.scss`，其中 `.td-home .td-outer { min-height: auto }` 用于消除首页 hero 与页脚之间的空白（主题默认 `min-height:100vh` + `.td-main{flex-grow:1}` 会在内容不足时把页脚顶到底部）。已删除 `docs/`、`book/` 栏目、`examples/` profile，以及被禁用的英法内容与 `i18n/fr.yaml`。Logo 在 `assets/icons/logo.svg`，favicon 在 `static/favicon.svg`。Giscus 评论已接入；`feedback`、`page_width`、Google Analytics 仍为注释。README 是上游模板文档，部分指向已删内容，以本文件与实际配置为准。

## 相关文档页

写内容前先查：`/zh/docs/start/anatomy/`（仓库地图）、`/zh/docs/customize/config/`（全部配置键）、`/zh/docs/customize/agents/`（`.md`/`llms.txt`/`navigation.json` 输出）、`/zh/docs/admin/deploy/`（托管配置）、`/zh/docs/admin/upgrade/`。
