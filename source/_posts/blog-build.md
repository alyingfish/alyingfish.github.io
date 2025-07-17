---
title: Hexo + GitHub Pages 搭建个人博客
date: 2025-07-18 02:00:48
index_img: 
categories:
  - 博客配置
tags: 
  - Hexo
  - Github Pages
  - 博客配置
math: true
---

> 基于 Hexo 框架和 Fluid 主题搭建静态博客，并部署在 GitHub Pages 上。

<!-- more -->

## 写博客的原因

人脑善于思考，但不擅长记忆。

智慧能够创造知识，但不能将知识长久的保存。

因此文字的发明才如此重要，因为从此人们就能专注于思考，专注于创造新的知识。而由文字来负责知识的保存、分享和继承。

把能够背诵许多诗词的孩子称为“聪明”，但充其量只是培养一个“数据库”而言。“数据库”不能思考，不能创造新的知识。

有意思的是，目前的AI大模型也严重缺乏长时间的记忆能力，需要人们给它适配各种“知识库”、“上下文”。

人类难道要抛弃目前“最优秀”的思考能力，去和计算机卷“数据库”的位置吗？

## 要写的东西

1. 知识的整理，方便回顾。
2. 思考的记录。
3. 脑洞、感想。

## 写博客的要求

1. 必须要能够给别人看，能够分享、传递的东西才是有生命的东西。
2. 应该能在多个设备上写博客，而非一定在电脑上。
3. 文章应易于书写和渲染。
4. 最好能够实时预览博客渲染的效果。
5. 不应该浪费太多时间在搭建博客网站上。
6. **没有审查**。

## 博客搭建准备

搭建个人博客一般需要这些：

- 文章。
- 博客框架。用于渲染博客网页。
- 服务器。提供 Web 服务。

按照`不应该浪费太多时间在搭建博客网站上`的要求，选择：

- 文章格式选择 Markdown 文档。
- 支持 Markdown 文档的静态博客框架，有丰富的主题，能开箱即用。
- 博客部署在 [Github Pages](https://pages.github.com/) 上，其提供免费的静态网页托管服务。

这里选择 [Hexo](https://hexo.io/) 作为博客框架，并使用 [Fluid 主题](https://github.com/fluid-dev/hexo-theme-fluid)，只需少量的配置，就能得到一个美观且功能全面的博客。

手机上可以通过在 github 仓库内上传/修改 markdown 文件来上传/修改博客文章。每次 commit 后，Github Pages 会重新渲染博客。

## 开始搭建

### 安装必要软件

对于 Archlinux 运行以下命令。

```bash
sudo pacman -S nodejs git
sudo npm install -g hexo-cli
```

详细请查看官方的 [Hexo文档 Installation](https://hexo.io/docs/#Installation)

### 建立博客网站

运行这些命令来创建一个 hexo 博客。

```bash
hexo init <folder>
cd <folder>
npm install
```

运行以下命令并打开 <http://localhost:4000/> 即可预览博客网站：

```bash
hexo generate
hexo server
```

详细请查看[Hexo文档 Setup](https://hexo.io/docs/setup)。

### 安装博客主题

推荐 [fluid](https://github.com/fluid-dev/hexo-theme-fluid) 开箱即用。只需简单的配置便能得到一个美观且功能全面的博客。

运行下面命令来安装 fluid 主题。

```bash
npm install --save hexo-theme-fluid
cp node_modules/hexo-theme-fluid/_config.yml _config.fluid.yml
```

在 `_config.yml` 中将主题设置为 fluid。

```yaml
theme: fluid  # 指定主题
language: zh-CN  # 指定语言，会影响主题显示的语言，按需修改
```

首次使用主题的「关于页」需要手动创建：

```bash
hexo new page about
```

创建成功后，编辑博客目录下 `source/about/index.md`，添加 `layout` 属性。

修改后的文件示例如下：

```markdown
---
title: about
layout: about
---

这里写关于页的正文，支持 Markdown, HTML
```

之后运行以下命令并打开<http://localhost:4000/>，来预览fluid主题：

```bash
hexo clean
hexo g
hexo s
```

安装指南详细见 [fluid 快速开始](https://github.com/fluid-dev/hexo-theme-fluid#%E5%BF%AB%E9%80%9F%E5%BC%80%E5%A7%8B)。

### 卸载默认主题

不需要默认主题，可以运行下面命令来卸载：

```bash
npm uninstall hexo-theme-landscape --save
rm _config.landscape.yml
```

## 个性化设置

博客已经搭建完成，但个人信息都是默认的，一些配置也需要修改。

### Hexo 博客配置

修改 `_config.yml` 文件。这里只显示修改了的部分：

{% fold info @_config.yml %}

```yaml config.yml
# Site
title: Lying Fish 的博客
subtitle: Lying Fish 的博客
description: (╯°□°)╯不要偷懒
author: Lying Fish
language: zh-CN
timezone: "Asia/Shanghai"

# URL
url: https://alyingfish.github.io/
root: /
permalink: /posts/:title/

# Writing
post_asset_folder: true
marked:
  prependRoot: true
  postAsset: true

# Extensions
theme: fluid
```

{% endfold %}

再为 `scaffolds/post.md` 添加：

- `category`，分类
- `tag`，标签
- `index_img`，缩略图
- `banner_img`，文章顶部大图
- 摘要，`<!-- more -->` 上面的内容。

{% fold info @scaffolds/post.md %}

```markdown
---
title: {{ title }}
date: {{ date }}
categories:
tags:
index_img:
banner_img:
---
这是摘要，最多显示三行。
<!-- more -->
```

{% endfold %}

### fluid主题配置

主题配置详细请查看 [Hexo Fluid 用户手册-配置指南](https://hexo.fluid-dev.com/docs/guide/)

修改 `_config.fluid.yml` 文件。这里只显示修改了的部分：

{% fold info @_config.fluid.yml %}

```yaml
code:
  highlight:
    line_number: false
    highlightjs:
      # 在链接中挑选 style 填入
      # Select a style in the link
      # See: https://highlightjs.org/demo/
      style: "github"
      style_dark: "github-dark-dimmed"

navbar:
  # 导航栏左侧的标题，为空则按 hexo config 中 `title` 显示
  # The title on the left side of the navigation bar. If empty, it is based on `title` in hexo config
  blog_title: "Lying Fish"

footer:
  statistics:
    enable: true

index:
  banner_img: /img/default.jpg
  slogan:
    text: "最短的捷径就是绕远路"

post:
  banner_img: /img/default.jpg
  footnote:
    header: '<h2>参考</h2>'

archive:
  banner_img: /img/archive.jpg

category:
  banner_img: /img/category.jpg

tag:
  banner_img: /img/tag.jpg

about:
  banner_img: /img/about.jpg
  avatar: /img/my-avatar.png
  name: "Lying Fish"
  intro: "不会游泳"
  icons:
    - { class: "iconfont icon-github-fill", link: "https://github.com/alyingfish", tip: "GitHub" }
```

{% endfold %}

配置中 `/img/*.jpg` 的图片需要手动添加到博客目录下的 `source/img/` 目录中，否则会使用主题默认的图片。

> 这个博客的页面顶部大图均来自艺术家 [Artem Chebokha](https://www.artstation.com/rhads)

## 部署到GitHub Pages

详细请查看 [Hexo 部署指南](https://hexo.io/docs/github-pages)。

按指南创建一个 `.github/workflows/pages.yml` 文件，并将指南中的内容粘贴进去，记得修改`nodejs` 版本号。

这个文件的作用是配置 GitHub Actions 来自动部署到 GitHub Pages。

确认 `.gitignore` 文件是否包含 `Public/` 目录，如无需手动添加，里面是生成的静态资源文件。`.gitignore` 应已带了：

```
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
_multiconfig.yml
```

之后在 Github 上创建一个 `<username>.github.io` 的仓库。运行下面命令来把本地博客仓库同步到 Github:

```bash
git init
git add .
git commit -m "first commit"
git branch -M main  # 更改当前branch名字为main
git remote add origin git@github.com:<username>/<username>.github.io.git # 替换 <username> 为 github 账户名
git push -u origin main
```

最后在 GitHub 下的博客仓库设置：`Settings` -> `Pages` -> `Build and deployment` -> `Source` 为 `GitHub Actions`。

之后每次 commit，GitHub Actions 都会自动运行部署脚本，渲染静态网页并部署到 GitHub Pages。

确认 github pages 部署完成后，访问 `https://<username>.github.io/` 即可看到博客。

## 启用评论插件

在 `_config.fluid.yml` 中配置评论插件：

```yml
post:
  comments:
    enable: true
    type: giscus

giscus:
  repo: <username>/<username>.github.io
  repo-id: # 填入 Giscus 生成的参数
  category: Announcements
  category-id: # 填入 Giscus 生成的参数
  theme-light: light
  theme-dark: dark_dimmed
  mapping: pathname
  reactions-enabled: 1
  emit-metadata: 0
  input-position: top
  lang: zh-CN
```

这里使用 [Giscus](https://github.com/giscus/giscus)作为评论插件。开源、免费、无广且数据都在 Github 上。

giscus 基于 Github Discussions，因此和基于 Issues 的评论插件不同，支持嵌套回复。

缺点是回复必需要认证 GitHub 账号。

### 配置并启用Giscus

要启用 Giscus，对应的博客仓库要启用 Discussions 功能，并安装 [giscus app](https://github.com/apps/giscus)。

查阅 [Github文档](https://docs.github.com/en/discussions/quickstart)以了解如何启用 Discussions。

同样，查看 [giscus app页面](https://github.com/apps/giscus) 并点击安装安装，Repository access 选择对应的博客仓库即可。

在 [giscus主页](https://giscus.app) 确认仓库符合要求，并完成剩下的设定。

最后，得到输出的 `<script>` 后，将其中对应的内容填写到上面的 `fluid` 的 giscus 配置中。

## 启用LaTeX数学公式

如何启用 $\LaTeX$ 数学公式，请查阅 [Hexo Fluid 用户手册-LaTex数学公式](https://fluid-dev.github.io/hexo-fluid-docs/guide/#latex-%E6%95%B0%E5%AD%A6%E5%85%AC%E5%BC%8F)。

为能够同时使用 markdown 的图片引用语法——`![](foo/bar.jpg)`，这里选择 `hexo-renderer-markdown-it` + `katex`。

关于如何使用使用 `![](foo/bar.jpg)` 来引用图片（编辑器可以预览图片，同时Hexo能够正确渲染图片），可见[这篇博文](../hexo-img)

公式测试：

行内公式：$T_n = 2T_{n-1} + 1 = 2(2^{n-1} - 1) + 1 = 2^n - 1$

独立公式：

$$S = \prod_{k=0}^{m} \frac {1 - p_k^{a_{k+1}}} {1-p_k}$$

## 使用 Tag 插件

fluid 主题自带一些便签插件，相比单纯的引用 `>` 要更美观些，见 [Tag 插件](https://hexo.fluid-dev.com/docs/guide/#tag-%E6%8F%92%E4%BB%B6)。

在 markdown 中加入如下的代码来使用便签：

```md
{% note success %}
文字 或者 `markdown` 均可
{% endnote %}
```

或者使用 HTML 形式：

```html
<p class="note note-primary">标签</p>
```

可选便签有：primary, secondary, success, danger, warning, info, light

fluid 主题的便签，里面的超链接文字颜色设定成了文本的颜色，我喜欢更有区分度。

在 fluid 主题 Tag 的 css 文件 `hexo-theme-fluid/source/css/_pages/_post/post-tag.styl` 中，将对应的一行注释掉：

```css
// note
.note
  a
    // color var(--text-color)
```
