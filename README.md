# Notebooks

基于 [Hexo](https://hexo.io/) 和 [NexT](https://theme-next.js.org/) 的个人博客 / 笔记站点，发布在 GitHub Pages。

## 本地开发

```bash
pnpm install
pnpm run build
pnpm run server
```

预览地址通常为 <http://localhost:4000/>。

## 常用命令

```bash
pnpm run clean
pnpm run build
pnpm run server
```

## 发布

项目通过 `.github/workflows/pages.yml` 在推送 `main` 分支时自动构建 `public/` 并部署到 GitHub Pages。

## 目录结构

```text
source/_posts/     文章
scaffolds/         新建文章模板
themes/            主题目录
_config.yml        Hexo 配置
_config.next.yml   NexT 配置
```
