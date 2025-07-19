---
title: Hexo 使用 Markdown 引用图片并支持预览
date: 2025-06-29 16:56:09
index_img:
categories:
  - 博客配置
tags:
  - Hexo
  - Markdown
  - 图片引用
---
Hexo 使用 Markdown 引用图片的语法通过相对路径来引用本地图片，以使得编辑器支持预览图片。

<!-- more -->

本文的希望达成的目的是：

1. 每个 Hexo 博客文章都能有独立的图片资源文件夹。
2. 通过相对路径来引用图片，并且 markdown 编辑器能够预览图片
3. Hexo 生成的 HTML 能够正确渲染图片。

## 启用专属资源文件夹

Hexo 统一将资源存放在 `source/image` 中，博客文章的图片资源也可以放在这里，但如果你想将每篇文章的资源单独存放在一个文件夹中，可以在 Hexo 配置文件 `_config.yml` 中启用 `post_asset_folder` 选项：

```yml
post_asset_folder: true
```

启用该选项后，使用 `hexo new <title>` 生成新的博客文章后，就会同时创建一个和 `<title>` 同名的目录，里面可以放置该文章的图片等资源，你也可以手动创建该目录。

```
source/_posts
├── foo.md
└── foo/
    └── image.jpg
```

这里默认开启该选项，并通过相对路径来引用图片。

## 官方的 Tap Plugins 图片引用方式

在 Hexo 中，官方推荐的[引用图片](https://hexo.io/docs/tag-plugins#Embed-image)的方式是：

```markdown
{% asset_img image.jpg %}
```

生成的 HTML 如下：

```html
<img src="/2020/01/02/foo/image.jpg">
```

但 markdown 编辑器无法识别该语法 Tag Plugins 语法，也就无法预览图片。

因此首先排除该方法。

## markdown 自带的图片引用方式

也就是最常见的引用方式：

```markdown
![](./foo/image.jpg)
```

markdown 编辑器可以完美预览图片，且支持的话，路径也能自动补全。

但 Hexo 不能正确解析该路径，生成的 HTML 如下：

```html
<img src="/./foo/image.jpg">
```

正确的路径应该是：

```html
<img src="/2020/01/02/foo/image.jpg">
```

导致图片无法显示。

## 不太完美的解决方法

一种解决方法是：干脆将文章路径中的日期删去，让 Hexo 生成的 HTML 能够正确解析到图片

```yml
permalink: :year/:month/:day/:title/
# 将上面改为：
permalink: /:title/
```

这样一来，正确的 HTML 就会是：

```html
<img src="/foo/image.jpg">
```

经过这种方式，Hexo 能够正确解析图片路径，markdown 编辑器也能够预览图片。

但有一个缺点，文章全部放在根目录下，数量一多，网页目录就会变得很乱，

## 使用 hexo 默认渲染器解析 markdown 图片语法

hexo 的默认渲染器 [hexo-renderer-marked](https://github.com/hexojs/hexo-renderer-marked) 3.1.0 引入了一个新选项，允许使用 markdown 中嵌入图像。

见：<https://hexo.io/docs/asset-folders#Embedding-an-image-using-markdown>

需要启用：

```yml
post_asset_folder: true
marked:
  prependRoot: true
  postAsset: true
```

能够将 `![](image.jpg)` 或 `![](./image.jpg)`，解析为：`<img src="/2020/01/02/foo/image.jpg">`

### 存在的问题

你可能注意到了，上面的图片路径 `image.jpg` 并不是相对引用，因为 .md 文件和图片并不在同一个文件夹。
因此 markdown 编辑器实际并不能够预览该图片。

如果使用图片路径 `![](foo/image.jpg)`，渲染器又无法解析了。

这是因为 hexo-renderer-marked 的原理是：

1. 找到文章 `foo.md` 同目录下的对应资源目录 `foo/`。
2. 在该目录下 `foo/`，搜索图片路径 `image.jpg` 或者是 `foo/image.jpg`
3. 找到该图片后，将其路径解析为 html。

显然 `image.jpg` 能够在目录 `foo/` 下找到，而 `foo/image.jpg` 无法在 `foo/` 目录下找到。

### 解决方法

关于这个问题，Github 上已有讨论，见 [hexo-renderer-marked #216](https://github.com/hexojs/hexo-renderer-marked/issues/216)。

并有了一个 [PR 271](https://github.com/hexojs/hexo-renderer-marked/pull/271)，但到现在 **2025-06-29**，该 PR 尚未合并。因此，如果你需要解决该问题，请根据该 PR 修改 `renderer.js` 的代码。

经测试，根据 PR 修改代码后，`![](foo/image.jpg)` 和 `![](./foo/image.jpg)` 均能正确解析。

该 PR 的原理是：当同名目录 `foo/` 搜不到图片时，转而搜索其父目录 `foo/..` 。

## 更换另一个能渲染markdown图片语法的渲染器

除了 hexo 默认的 markdown 渲染器外，另一个渲染器 [hexo-renderer-markdown-it](https://github.com/hexojs/hexo-renderer-markdown-it) 同样允许使用 markdown 中嵌入图像。

这个渲染器要比默认的渲染器更快，而且可以[通过插件来使用 LaTex 公式](https://fluid-dev.github.io/hexo-fluid-docs/guide/#latex-%E6%95%B0%E5%AD%A6%E5%85%AC%E5%BC%8F)，因此推荐这一种方法。

### 安装并配置 `hexo-renderer-markdown-it`

首先需要卸载默认渲染器，并安装该渲染器：

```bash
npm uninstall hexo-renderer-marked --save
npm install hexo-renderer-markdown-it --save
```

`hexo-renderer-markdown-it` 同样实现了 `prependRoot` 和 `postAsset` 参数，但设置有所不同。

在**站点配置**中，`_config.yml` 中设置：

```yml
post_asset_folder: true
# 注释掉默认渲染器的设置选项
# marked:
#   prependRoot: true
#   postAsset: true
markdown:
  images:
    prepend_root: true
    post_asset: true
  # 下面的插件为 latex 的插件，可删除
  plugins:
    - "@traptitech/markdown-it-katex"
```

`hexo clean` 后，重新生成并预览博客。

### 一点小缺憾

`hexo-renderer-markdown-it` 渲染器可以解析 `![](foo/bar.svg)`，但无法解析 `![](./foo/bar.svg)`，见 [Issue # 220](https://github.com/hexojs/hexo-renderer-markdown-it/issues/220)
