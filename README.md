# GitHub Pages 网页托管说明

GitHub Pages 是 GitHub 提供的 **静态网站托管服务**，可以直接把仓库里的 HTML/CSS/JS（或 Jekyll 生成的站点）发布为公开可访问的网址，免费、无需自建服务器。

---

## 一、GitHub Pages 的两种站点

| 类型 | 仓库名 | 访问地址 | 用途 |
| --- | --- | --- | --- |
| 用户/组织页 | `<用户名>.github.io` | `https://<用户名>.github.io` | 个人主页、博客 |
| 项目页 | 任意仓库名 | `https://<用户名>.github.io/<仓库名>` | 具体项目的文档站 |

> 用户/组织页每个账号只能有一个；项目页不限制数量。

---

## 二、快速开始（三步发布）

### 方式 A：从仓库设置直接发布（最简单）

1. 在 GitHub 新建仓库（用 `<用户名>.github.io` 可作个人主页）。
2. 把网页文件（`index.html` 等）推送到仓库。
3. 进入仓库 **Settings → Pages**：
   - **Source（源）**：选 `Deploy from a branch`
   - **Branch（分支）**：选 `main`（或 `master`）
   - **Folder（目录）**：选 `/ (root)` 或 `/docs`
   - 点击 **Save**
4. 等待约 1 分钟，访问 `https://<用户名>.github.io/<仓库名>` 即可。

### 方式 B：用 GitHub Actions 自动构建发布（推荐用于 Jekyll/Hugo/构建型项目）

`.github/workflows/pages.yml` 示例：

```yaml
name: Deploy to GitHub Pages
on:
  push:
    branches: [main]
permissions:
  contents: read
  pages: write
  id-token: write
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      # 如需构建步骤（如 npm run build）在此添加
      - uses: actions/upload-pages-artifact@v3
        with:
          path: ./dist   # 构建产物目录
  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/deploy-pages@v4
```

发布后可在 **Settings → Pages** 看到绿色的 "Your site is live" 提示。

---

## 三、使用 Jekyll（GitHub 原生支持）

仓库根目录放 `_config.yml` 即可启用 Jekyll 主题，无需手动构建：

```yaml
title: 我的站点
description: 用 GitHub Pages 搭建
theme: jekyll-theme-minimal
markdown: kramdown
```

- 提交 `.md` 文件会自动渲染成 HTML。
- 想禁用 Jekyll（用纯静态文件或自己的构建工具）：在根目录放一个空文件 `.nojekyll`。

---

## 四、自定义域名 + HTTPS

1. 在域名服务商处添加记录：
   - 用 APEX 域名（如 `example.com`）：添加 4 条 A 记录指向
     `185.199.108.153` / `185.199.109.153` / `185.199.110.153` / `185.199.111.153`
   - 用子域名（如 `www.example.com`）：添加 CNAME 记录指向 `<用户名>.github.io`
2. 在 **Settings → Pages → Custom domain** 填入你的域名，保存。
3. 勾选 **Enforce HTTPS**（需等待证书签发，通常几分钟到几小时）。
4. 项目根目录放 `CNAME` 文件，内容为你的域名（GitHub 通常自动生成）。

---

## 五、本地预览（避免反复提交调试）

```bash
# 需要 Ruby 环境
gem install bundler jekyll
bundle init
bundle add jekyll
bundle exec jekyll serve   # 默认 http://localhost:4000
```

纯静态站点可直接用任意本地服务器，例如：

```bash
python -m http.server 8000
```

---

## 六、重要限制（免费版）

- **仅支持静态内容**：不能运行后端、数据库、服务端脚本（PHP/Python 后端等）。
- 仓库软上限约 **1 GB**，单个文件 ≤ 100 MB。
- 站点带宽约 **100 GB / 月**，构建次数约 **10 次 / 小时**。
- 页面请求会经 CDN 缓存，更新可能有短暂延迟。

---

## 七、常见问题排查

| 现象 | 可能原因 / 解决 |
| --- | --- |
| 404 页面 | 没放 `index.html`；或 Pages 源分支/目录选错 |
| 样式丢失 | 使用了绝对路径 `/assets/...`，改用相对路径或设 `baseurl` |
| 自定义域名 HTTPS 不生效 | 等待证书签发；确认 DNS 记录正确且已保存域名 |
| 改了代码没更新 | 清缓存刷新；Actions 构建是否失败（看 Actions 日志） |
| 中文乱码 | `index.html` 加 `<meta charset="utf-8">` |

---

## 八、目录示例（项目页）

```
my-site/
├── index.html
├── assets/
│   ├── style.css
│   └── script.js
├── _config.yml        # 使用 Jekyll 时
├── .nojekyll          # 纯静态/自构建时
├── CNAME              # 自定义域名时
└── .github/
    └── workflows/
        └── pages.yml  # Actions 发布
```

---

> 提示：更完整的官方文档见 <https://docs.github.com/pages>。
