# CLAUDE.md — nexus-docs

本文件为在此仓库中工作的 Claude Code 提供指引。

## 项目概述

`nexus-docs` 是 **Hi168 云平台的在线帮助文档站**，基于 [docsify](https://docsify.js.org/) 的纯静态站点，无构建步骤。

- 线上入口：`https://www.hi168.com/help/`
- 网关链路：`nexus-gateway` 的 `/help/*` → `nexus-docs:80`（网关环境变量 `NEXUS_HELP_ADDR`）
- 生产 K8s：`hi168-platform` 命名空间下的 `nexus-docs` Deployment / Service
- 仓库：`git@github.com:hwua-hi168/nexus-docs.git`（`origin/main`）
- 版本号：`VERSION`（当前 `v2.10.0`）

> ⚠️ 本仓库只负责「在线帮助文档」，不含平台代码。平台代码在各 `nexus-*` 仓库。

## 目录结构

```
nexus-docs/
├── index.html          # docsify 配置 + 图片放大模态框 + 侧边栏折叠插件
├── home.md             # 帮助站首页（默认加载页）
├── _sidebar.md         # 根级侧边栏（首页使用）
├── _navbar.md          # 顶部导航（当前基本为空/注释）
├── vue.css / docsify.min.js
├── docs/               # ★ 当前版本的帮助文档正文
│   ├── README.md           # 文档导览
│   ├── _sidebar.md         # docs/ 下页面的侧边栏（与根级不同，需同步维护）
│   ├── application-services.md
│   ├── template-image.md
│   ├── workspace.md
│   ├── storage.md
│   ├── online-learning.md
│   ├── teaching-management.md
│   ├── topology.md
│   ├── ai-playground.md
│   ├── notebook.md
│   ├── dataset-model.md
│   ├── co-creation.md
│   ├── community.md
│   ├── organization.md
│   ├── platform-settings.md
│   ├── microservices.md
│   ├── deployment.md
│   ├── appendix.md
│   └── images/help/        # 当前版本界面截图（41 张）
├── manual/             # ⚠️ 旧版手册（旧 UI + 220 张老截图），新文档不要写这里
├── Dockerfile          # nginx:alpine + 静态文件
├── nginx.conf          # SPA 回退 + 静态缓存 1h + gzip
├── taskfile.yaml       # image / push / release / dev
└── VERSION
```

## 本地预览

```bash
# 方式一：任意静态服务器（最稳，不依赖 npx）
python3 -m http.server 8099 --bind 127.0.0.1
# 打开 http://127.0.0.1:8099/#/docs/application-services

# 方式二：docsify-cli
npx docsify-cli serve . --port 3000
# 或 task dev
```

首页：`http://127.0.0.1:8099/#/`（加载 **`home.md`**，其上的「完整文档导航」指向 `docs/README.md`）。

> `index.html` 里的 `homepage` 会根据 URL 查询参数变化：`?docs/xxx` 加载 `docs/xxx.md`，`?manual/01/xxx` 加载旧手册；无参数时默认 `home.md`。正常的 hash 路由（`#/docs/xxx`）不受影响。
>
> ⚠️ 查询参数式链接（`?docs/xxx`）加载的页面处于根路由 `/`，其内容里的相对链接/图片会按根目录解析而失效；站内一律用 hash 路由 `#/docs/xxx` 链接。

## 内容约定（新增/修改文档必读）

### 1. 写到哪里

| 内容 | 位置 |
| --- | --- |
| 当前版本的帮助文档 | `docs/*.md` |
| 当前版本截图 | `docs/images/help/*.png` |
| 首页/导览 | `home.md`、`docs/README.md` |
| 旧版手册 | `manual/` —— **只读，不要新增或“顺手更新”** |

### 2. 每篇文档的结构

统一按以下顺序组织，便于用户对照界面操作：

```
# 模块名
## 概述
## 一、入口（桌面图标 / 启动菜单 / 控制台路径）
## 二、界面说明（放截图）
## 三、操作步骤 / 字段说明（表格）
## 四、注意事项
## 五、常见问题（FAQ 表格）
```

要求：

- **用界面上的真实名称**：如「课程学习」「应用模板」「我的镜像」「教务管理」，不要自造名称。
- **先截图后写步骤**：步骤必须与当前 UI 一致，不能凭旧文档推断。
- **截图放在相关小节内**，不要全部堆在文末；每张图配一句说明（图片 alt 文本即图注）。
- 表格用于字段/筛选/FAQ；步骤用有序列表；提示/警告用 `>` 引用块。
- 价格、规格等易变数字注明「以页面实际显示为准」。

### 3. 链接与图片路径

> ⭐ `index.html` 的 docsify 配置里有 **`relativePath: true`**（见 docsify「Relative path」配置）。它让 md 里的相对链接/图片按**当前页面所在目录**解析。**不要删除这行**——删掉后全站 `./xxx.md` 链接会变成 `#/./xxx` 并报「链接发生错误」。

- 图片：`![说明](./images/help/xxx.png)`（相对 **当前 md 文件所在目录**）
- 同目录文档互链：`[计费规则](./appendix.md)`（docsify 会自动去掉 `.md`）
- 从 `docs/` 页跳到根级页面：`[返回首页](../home.md)`
- 从根级页面（`home.md`、`docs/README.md` 之外的根文件）跳到文档：`[应用服务](docs/application-services.md)`
- 侧边栏链接 **不带 `.md`**（docsify 路由约定），并相对侧边栏文件自身目录：
  - 根级 `_sidebar.md`：`[应用服务](./docs/application-services)`
  - `docs/_sidebar.md`：`[应用服务](./application-services)`
- ⚠️ 不要在 `docs/*.md` 里写 `./docs/xxx.md`（会解析成 `docs/docs/xxx.md`）；根级文件不要写 `./xxx.md` 指向 docs 内页面。
- ✅ 改完链接用本地预览逐页点一遍（或跑 `#/`、`#/docs/README`、任一 `#/docs/xxx` 三个入口的侧边栏），确认 URL 形如 `#/docs/online-learning`，出现 `#/./xxx` 即为配置被改坏。

### 4. 侧边栏

- 根级 `_sidebar.md` 与 `docs/_sidebar.md` **是两份不同的文件**，分别服务首页与 docs 页面；增删页面时 **两处都要改**。
- 分组使用 `* **分组名**`，docsify 插件会自动折叠子项。

### 5. 截图规范

- 分辨率统一 **1440×900**（Playwright 视口），窗口最大化后截图。
- 教育类页面在**本地开发环境**实测截图（`localhost:9090`）。
- 其他模块在**生产环境只读浏览**截图（`www.hi168.com`，教师账号）；**禁止在生产创建/删除任何资源**。
- 截图前清除敏感信息：账号名、手机号、真实姓名；无法清除时需打码或换测试账号。
- 文件名用小写英文/短横线：`app-center.png`、`course-play.png`、`eduadmin-class.png`。
- 截图文件放在 `docs/images/help/`，不要放 `manual/help_picture/`。

## 构建与发布

### 镜像构建

```bash
task image        # docker build -t registry.cn-hangzhou.aliyuncs.com/quantanexus/help-docs:$(cat VERSION) .
task push         # 推送到阿里云 ACR
task release      # image + push
```

> `taskfile.yaml` 中 `IMAGE_NAME: help-docs`，但生产 Deployment 名为 **`nexus-docs`**、镜像为 `.../nexus-docs:vX.Y.Z-BUILD`。发布前务必核对 Jenkins job 实际推送的镜像名与 tag，不要只照搬 taskfile。

### 生产发布

生产由 Jenkins job 构建（参考 main 仓库的 `/publish-hi168` skill 与 `docs/publish-hi168-harbor-runbook.md`），完成后：

```bash
ssh root@192.168.6.147
kubectl -n hi168-platform rollout status deploy/nexus-docs
kubectl -n hi168-platform get deploy nexus-docs -o jsonpath='{.spec.template.spec.containers[0].image}'
```

验证：打开 `https://www.hi168.com/help/`，确认新页面/截图可见（nginx 对 md/png 缓存 1 小时，必要时 `Ctrl+F5`）。

## 常见坑

1. **文档缓存**：nginx 对 `.md/.png` 设置 `expires 1h`；发布后看不到更新先强刷或换无痕窗口。
2. **首页被查询参数劫持**：`index.html` 的 `homepage` 逻辑依赖 `?docs/...`；带查询参数访问会加载不同首页（且因处于根路由，页内相对链接/图片会失效），排查“首页不对 / 图片 404”时先看 URL 有没有 `?`。
3. **`relativePath` 被删导致全站链接 404**：`index.html` 中 `relativePath: true` 必须保留；现象是所有链接变成 `http://…/#/./online-learning` 并显示「链接发生错误o(╯□╰)o」。
4. **首页文件必须在根目录**：docsify 在 `#/` 路由下按根目录解析相对路径，所以首页是根级 `home.md`（不是 `docs/README.md`）；`docs/README.md` 只作为 `#/docs/README` 的文档导览页。
5. **两份 sidebar**：只改根级或只改 `docs/_sidebar.md`，会出现某些页面侧边栏缺项。
6. **图片路径**：图片相对 md 文件解析；从别的目录复制内容时最容易出现 404。
7. **旧 manual 混入**：`manual/` 是旧 UI 手册，截图与当前版本不一致；新文档只写 `docs/`。
8. **不要改 `index.html` 的 docsify 插件逻辑**（折叠/图片放大）除非明确要求，改动会影响全站。
9. **无构建步骤**：不要引入打包工具；docker 镜像就是静态文件 + nginx。

## 相关文档

- 本次扩充的记录与复跑说明：`../../tasks/task-help-docs-expand/README.md`
- Word 审核稿：`../../tasks/task-help-docs-expand/Hi168-新版帮助文档扩充-审核稿.docx`
- 发布 runbook：`/mnt/app/hwua/docs/publish-hi168-harbor-runbook.md`
- 平台环境与发布速览：`../当前项目环境与发布-操作速览.md`