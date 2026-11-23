---
name: quartz-github-pages
description: Use this skill when the user wants to publish a folder of Markdown notes (e.g. an Obsidian vault, a personal wiki, or project docs) as a public website using Quartz and GitHub Pages — including enabling GitHub Pages, setting up GitHub Discussions, and adding giscus comments. Covers the full path from an existing Markdown folder to a live, commentable site.
---

# Quartz + GitHub Pages 发布教程

把一个本地 Markdown 文件夹（Obsidian vault、个人知识库、项目文档等）发布成一个公开网站，使用 [Quartz](https://quartz.jzhao.xyz/) 作为静态站点生成器，GitHub Pages 作为免费托管，giscus 作为评论系统。

本教程基于真实走通的一次完整流程整理，包含实际遇到的坑和排错方式，不是照搬官方文档的摘要。

## 前置条件

- Node.js >= 22（`node -v` 检查）
- Git
- 一个 GitHub 账号
- 本地到 `github.com` 的网络连通性（见下方「常见问题」）

## 整体流程

1. 用 Quartz 脚手架搭建站点骨架，把已有 Markdown 内容接入
2. 本地预览验证
3. 推送到 GitHub 仓库
4. 启用 GitHub Pages
5. 启用 GitHub Discussions + 配置 giscus 评论

---

## 第一步：GitHub 账号与 SSH 认证

如果还没有和 GitHub 的 SSH 认证，先配置好，否则后面推不上去。

```bash
# 生成一个新的 SSH key（用你 GitHub 账号绑定的邮箱）
ssh-keygen -t ed25519 -C "你的邮箱@example.com" -f ~/.ssh/id_ed25519 -N ""

# 查看公钥内容，复制整行
cat ~/.ssh/id_ed25519.pub
```

把公钥内容添加到 **https://github.com/settings/ssh/new**，Title 随便填，粘贴公钥，保存。

验证连接：

```bash
ssh -T git@github.com
# 看到 "Hi <你的用户名>! You've successfully authenticated" 就说明成功
```

同时设置一下 git 身份（如果还没设置过）：

```bash
git config --global user.name "<你的 GitHub 用户名>"
git config --global user.email "<你的邮箱>"
```

---

## 第二步：获取 Quartz 并接入已有内容

假设你已有的 Markdown 内容在 `~/path/to/your-notes`（里面是一堆 `.md` 文件和子目录）。

### 2.1 克隆 Quartz 脚手架

```bash
git clone https://github.com/jackyzha0/quartz.git my-site
cd my-site
```

> 默认分支是 `v5`，GitHub Pages 的部署工作流也是以推送到 `v5` 分支为触发条件，不要改分支名，除非你知道自己在改什么。

### 2.2 安装依赖

```bash
npm i
```

> **常见问题**：如果遇到 `npm error EACCES` 或缓存权限报错（通常是之前某次用 `sudo npm` 导致缓存目录被 root 占用），不要用 `sudo` 硬修，改用隔离缓存目录重试：
> ```bash
> npm i --cache /tmp/npm-cache-quartz
> ```
> 之后所有 `npx quartz ...` 命令都加上 `npm_config_cache=/tmp/npm-cache-quartz` 环境变量前缀，保持一致。

安装过程中如果提示有包的 install script 被跳过（`@parcel/watcher`、`esbuild`、`fsevents` 等原生编译依赖），批准它们：

```bash
npm install-scripts approve @parcel/watcher esbuild fsevents --cache /tmp/npm-cache-quartz
```

### 2.3 非交互式初始化，接入已有内容

用 `quartz create` 的非交互模式，一条命令把已有 Markdown 文件夹接进来：

```bash
npx quartz create \
  --template obsidian \
  --strategy copy \
  --source ~/path/to/your-notes \
  --baseUrl <你的GitHub用户名>.github.io/<仓库名>
```

参数说明：

| 参数 | 作用 |
|---|---|
| `--template obsidian` | 用 Obsidian 模板，原生支持 `[[wikilink]]`、callout、mermaid 图等 Obsidian 语法。如果你的笔记没用这些语法，用 `--template default` 也可以 |
| `--strategy copy` | 把源文件夹内容**复制**进 Quartz 的 `content/` 目录，不改动你原始文件。也可以用 `symlink`（软链接，原始文件改动会自动同步），但更脆弱，建议先用 copy |
| `--source` | 你已有 Markdown 内容的绝对路径 |
| `--baseUrl` | 站点将部署到的 URL，不带 `https://`。个人站（`<user>.github.io`）或项目站（`<user>.github.io/<repo>`）两种格式都支持，要和下面建的仓库名对应 |

> **常见问题**：如果模板默认启用了某个你装不上的插件（例如 `@quartz-community/obsidian-plugin-excalidraw` 因为网络或权限装不上），直接打开 `quartz.config.yaml` 把对应的 plugin 条目删掉即可，不影响其他功能。删完重新跑一次：
> ```bash
> npm_config_cache=/tmp/npm-cache-quartz npx quartz plugin install --from-config
> ```

### 2.4 补一个首页

Quartz 的站点首页需要 `content/index.md`，如果你的内容里只有 `README.md` 没有 `index.md`，复制一份：

```bash
cp content/README.md content/index.md
```

### 2.5 本地预览

```bash
npm_config_cache=/tmp/npm-cache-quartz npx quartz build --serve
```

浏览器打开 `http://localhost:8080` 验证内容、中文、链接是否正常渲染。确认没问题后 Ctrl+C 停掉。

---

## 第三步：推送到 GitHub

### 3.1 建一个空仓库

打开 **https://github.com/new**，填仓库名（和上面 `--baseUrl` 里的仓库名一致），**不要**勾选自动生成 README/.gitignore/license（保持空仓库，否则推送时会冲突）。

### 3.2 清理不需要的上游文件（可选但推荐）

Quartz 克隆下来自带它自己项目的 CI、CHANGELOG、Issue 模板等，和你的站点无关，可以删掉保持仓库干净：

```bash
rm -rf .changeset CHANGELOG.md CODE_OF_CONDUCT.md Dockerfile docs LICENSE.txt
rm -f .github/workflows/build-preview.yaml .github/workflows/ci.yaml \
      .github/workflows/dependabot-automerge.yaml .github/workflows/deploy-preview.yaml \
      .github/workflows/deploy-v5.yaml .github/workflows/docker-build-push.yaml \
      .github/workflows/release.yaml
rm -rf .github/ISSUE_TEMPLATE .github/pull_request_template.md .github/FUNDING.yml
```

### 3.3 推送

```bash
git remote add origin git@github.com:<你的用户名>/<仓库名>.git
git add -A
git commit -m "Initial site setup"
git push -u origin v5
```

---

## 第四步：添加 GitHub Pages 部署工作流

在仓库根目录创建 `.github/workflows/deploy.yaml`：

```yaml
name: Deploy Quartz site to GitHub Pages

on:
  push:
    branches:
      - v5

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 0 # Fetch all history for git info
      - uses: actions/setup-node@v6
        with:
          node-version: 24
      - name: Cache dependencies
        uses: actions/cache@v5
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-
      - name: Cache Quartz plugins
        uses: actions/cache@v5
        with:
          path: .quartz/plugins
          key: ${{ runner.os }}-plugins-${{ hashFiles('quartz.lock.json') }}
          restore-keys: |
            ${{ runner.os }}-plugins-
      - name: Install Dependencies
        run: npm ci
      - name: Install Quartz plugins
        run: npx quartz plugin install
      - name: Build Quartz
        run: npx quartz build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: public

  deploy:
    needs: build
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

提交并推送这个文件。

```bash
git add .github/workflows/deploy.yaml
git commit -m "Add GitHub Pages deploy workflow"
git push origin v5
```

## 第五步：开启 GitHub Pages

这一步必须在网页上手动操作，没有 API 能代替：

1. 打开 `https://github.com/<你的用户名>/<仓库名>/settings/pages`
2. 在 "Build and deployment" → "Source" 下拉菜单，选择 **GitHub Actions**

选完之后，之前推送触发的那次 workflow 可能已经跑过并在 deploy 步骤失败了（因为那时 Pages 还没开），这是正常的——推一个新 commit（哪怕是空提交）重新触发一次即可：

```bash
git commit --allow-empty -m "Trigger deploy"
git push origin v5
```

查看部署状态：`https://github.com/<你的用户名>/<仓库名>/actions`

部署成功后，站点会上线在：
- 项目站：`https://<你的用户名>.github.io/<仓库名>/`
- 个人站（仓库名为 `<用户名>.github.io`）：`https://<你的用户名>.github.io/`

---

## 第六步：启用评论功能（giscus）

giscus 把评论存在你仓库的 GitHub Discussions 里，免费、无需后端。

### 6.1 开启 Discussions

打开 `https://github.com/<你的用户名>/<仓库名>/settings` → Features → 勾选 **Discussions**

### 6.2 安装 giscus App

打开 **https://github.com/apps/giscus** → Install → 选择你的仓库授权

> 仓库必须是 **Public**，否则访客看不到评论。

### 6.3 生成配置

打开 **https://giscus.app**，按页面指引：
1. 输入你的仓库 `<你的用户名>/<仓库名>`
2. Discussion 分类选 **Announcements**
3. 其他选项按需调整（主题、语言等）
4. 页面底部会生成一段 `<script>` 配置，记下其中的 `data-repo-id` 和 `data-category-id` 两个值

### 6.4 写入 Quartz 配置

打开 `quartz.config.yaml`，找到 `@quartz-community/comments` 插件条目，改成：

```yaml
  - source: "@quartz-community/comments"
    enabled: true
    options:
      provider: giscus
      options:
        repo: <你的用户名>/<仓库名>
        repoId: <giscus.app 生成的 data-repo-id>
        category: Announcements
        categoryId: <giscus.app 生成的 data-category-id>
        mapping: pathname
        strict: false
        reactionsEnabled: true
        inputPosition: bottom
        lang: zh-CN
    layout:
      position: afterBody
      priority: 10
```

### 6.5 重新构建并推送

```bash
npm_config_cache=/tmp/npm-cache-quartz npx quartz plugin install --from-config
npm_config_cache=/tmp/npm-cache-quartz npx quartz build   # 本地验证评论区能渲染出来
git add quartz.config.yaml
git commit -m "Enable giscus comments"
git push origin v5
```

推送后等 Actions 跑完，刷新网站页面，底部就会出现评论区。

---

## 常见问题

### `curl`/`git clone` 连不上 github.com
有些网络环境对 GitHub 有速度限制或间歇性连接问题。先用 `curl -s -o /dev/null -w "%{http_code}\n" https://github.com` 测试连通性，如果超时可以重试几次，或者换成 `https://github.com/download` 之类官网重定向的地址再找真实下载链接。

### `npm i` 报 `EACCES` 或缓存权限错误
不要用 `sudo npm`。改用隔离缓存目录：`npm i --cache /tmp/npm-cache-quartz`，后续命令统一加 `npm_config_cache=/tmp/npm-cache-quartz` 环境变量。

### 某个模板默认插件装不上
直接编辑 `quartz.config.yaml` 删除对应的 `- source: "..."` 插件块，然后重跑 `npx quartz plugin install --from-config`。

### Wikilink（如 `[[某篇笔记]]`）点不过去
Quartz 按**文件名**解析 wikilink，不是按文章标题。如果链接文字和目标文件名不一致（比如中文标题链接到英文文件名的文件），需要在目标文件的 frontmatter 里加 `aliases` 字段：

```yaml
---
aliases:
  - 某篇笔记
---
```

### 部署后首页 404
检查 `content/` 目录下有没有 `index.md`（不是 `README.md`）。Quartz 的站点首页固定读取 `index.md`。

### 想用个人站（`<user>.github.io`）而不是项目站
仓库名必须正好是 `<你的用户名>.github.io`，`--baseUrl` 对应填 `<你的用户名>.github.io`（不带仓库名后缀）。

---

## 安装这个 Skill

```bash
git clone <此 Skill 所在仓库地址> ~/.claude/skills/quartz-github-pages
```

或者不用 git，直接下载 `SKILL.md` 这一个文件放到 `~/.claude/skills/quartz-github-pages/SKILL.md` 也可以用。
