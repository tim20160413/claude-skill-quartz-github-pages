# claude-skill-quartz-github-pages

一个 [Claude Code](https://claude.com/claude-code) Skill：把一个本地 Markdown 文件夹（Obsidian vault、个人知识库、项目文档等）发布成一个用 [Quartz](https://quartz.jzhao.xyz/) 构建、托管在 GitHub Pages 上的公开网站，包含开启 GitHub Pages、开启 GitHub Discussions、配置 giscus 评论的完整步骤。

内容基于一次真实走通的完整流程整理，包含实际遇到的坑（npm 缓存权限问题、插件装不上、wikilink 解析规则等）和排错方式，不是官方文档的摘要搬运。

## 安装

```bash
git clone https://github.com/tim20160413/claude-skill-quartz-github-pages.git ~/.claude/skills/quartz-github-pages
```

或者不用 git，直接下载 [`SKILL.md`](SKILL.md) 这一个文件，放到：

```
~/.claude/skills/quartz-github-pages/SKILL.md
```

安装后在 Claude Code 里描述你的需求（比如"帮我把这个 Markdown 文件夹发布成网站"），会自动匹配到这个 Skill。

## 内容概览

1. GitHub 账号与 SSH 认证配置
2. 用 Quartz 脚手架接入已有 Markdown 内容（非交互式命令）
3. 本地预览
4. 推送到 GitHub 仓库
5. 添加 GitHub Pages 部署工作流
6. 开启 GitHub Pages
7. 开启 GitHub Discussions + 配置 giscus 评论
8. 常见问题排错

## License

MIT
