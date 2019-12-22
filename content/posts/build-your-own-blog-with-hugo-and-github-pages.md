---
title: "使用 Hugo 和 GitHub Pages 搭建个人博客"
date: 2019-12-11T05:48:35+08:00
draft: true
toc: true
images: ["/img/build-your-own-blog-with-hugo-and-github-pages.png"]
tags: 
  - hugo
  - github-pages
  - github-actions
typora-root-url: ../../static
---

本文介绍如何使用 Hugo 快速构建个人博客站点, 并通过 Github Actions 自动化部署至 Github Pages.

## 准备清单

1. [Hugo](https://gohugo.io/): 快速便捷的静态站点生成工具.

    使用 [brew](https://brew.sh/) 安装

    ```sh
    brew install hugo
    hugo version
    ```

    本文使用的 `hugo` 版本是 `v0.60.1/extended`.

    其他安装方式见[这里](https://gohugo.io/getting-started/installing).

2. [git](https://git-scm.com/), 及其基本使用方式.
3. (可选) [hub](https://hub.github.com/): github 命令行工具, 我们将用它来创建 github 仓库.

    ```sh
    brew install hub
    ```

    其他安装方式见[这里](https://github.com/github/hub#installation).

## 使用 Hugo 构建站点

我们开始吧!

### 创建站点

```sh
hugo new site blog
```

`hugo` 将为我们创建一个名为 `blog` 的目录, 并初始化必要的结构和内容.

简单地看下 `blog` 目录的结构:

```txt
blog
├── archetypes
├── config.toml
├── content
├── data
├── layouts
├── static
└── themes
```

我们现在只需了解其中四个文件/目录的作用即可:

* `config.toml`: 站点配置文件
* `content/`: 之后创建的文章都会放在这里
* `static/`: 用于存放静态资源文件, 例如文章中使用的图片, Favicon 等. 发布时, `hugo` 会将在此目录下的所有文件均拷贝至站点根目录下.
* `themes/`: 用于存放我们使用的主题文件

### 挑选主题

Hugo 社区已为我们提供了很多漂亮的[主题](https://themes.gohugo.io/). 挑选一个你喜欢的主题, 然后我们把它安装到本地.

这里以我使用的 [hermit](https://themes.gohugo.io//hermit) 主题为例:

```sh
cd blog
git init
git submodule add https://github.com/Track3/hermit.git themes/hermit
```

主题文件会被添加到 `themes/<主题名>` 目录下.

### 配置

在 `themes/` 下每个主题均会有一个 `exampleSite/`, 我们把其中的示例配置文件 `config.toml` 拷贝到当前工作目录下

```sh
# 这里以 hermit 主题为例
cp themes/hermit/exampleSite/config.toml config.toml
```

调整一些必要的配置, 例如:

```toml
baseURL = "https://<username>.github.io/" # 你的 github pages 地址, 下文会再介绍
languageCode = "zh-cn" # 用于 RSS
title = "我的博客"
theme = "hermit" # 指定使用的主题
# ...
```

### 跑起来看看

`hugo` 可以作为一个 web 服务器启动, 并且能够实时载入更新的内容

```sh
hugo server
```

现在, 访问 <http://localhost:1313/> 就能在本地浏览我们的站点了.

![hugo-demo-home](/img/hugo-demo-home.png)

### 你的第一篇文章

我们用 markdown 来组织内容. 创建一篇文章:

```sh
hugo new posts/my-first-post.md
```

文件会在 `content/posts/` 下创建, 查看一下它的内容:

```markdown
---
title: "My First Post"
date: 2019-12-09T18:13:37+08:00
draft: true
toc: false
images:
tags:
  - untagged
---

```

最前面是一段 `YAML front matter`, 用于标注文件的元数据. 其中, `draft` 字段表示该文件是否处于 “草稿” 状态, 默认情况下, `draft: true` 的文章将不会包含在最终的发布文件中.

你看到的内容可能与上面的有些许不同, 这是根据主题目录(或当前目录)下 `archetypes/` 中的模板配置决定的.

试着在文章中添加一些内容, 然后将 `draft` 标记为 `false`.

我们已经可以在浏览器看到这篇文章了.

![hugo-demo-my-first-post](/img/hugo-demo-my-first-post.png)

> **Tips**: 如果你希望能够预览仍处于 “草稿” 状态的稿件, 可使用命令
>
> ```sh
> hugo server -D
> ```

### (可选) 制作 Favicon

为我们的站点制作一个个性化的 `Favicon`, 可以帮助我们更容易被读者记住. [RealFaviconGenerator.net](https://realfavicongenerator.net/).
只需在 RealFaviconGenerator.net 上传一张你喜欢的图片, 它就可以帮你生成覆各个平台所需的 Favicon 文件. 我们把这些文件拷贝到 `static/`

```txt
static
├── android-chrome-192x192.png
├── android-chrome-512x512.png
├── apple-touch-icon.png
├── browserconfig.xml
├── favicon-16x16.png
├── favicon-32x32.png
├── favicon.ico
├── mstile-150x150.png
├── safari-pinned-tab.svg
└── site.webmanifest
```

> **Tips**: 如果无法在浏览器中看到新的 Favicon, 多半是因为浏览器缓存引起的. 我们可以在启动 `hugo server` 时使用参数 `--noHTTPCache`, 禁止浏览器对内容缓存, 以便开发调试.
>
> ```sh
> hugo server --noHttpCache
> ```

### 选择一个合适的 LICENSE

知识共享协议

```sh
curl -L https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.txt > LICENSE
```

### 发布站点

如果一切看上去不错, 我们便可以开始构建站点. 这也是 `hugo` 最核心的命令

```sh
hugo --minify
```

> 参数 `--minify` 可以用于缩减 html 生成后的文件体积.

`hugo` 会创建一个 `public` 目录, 并将生成的所有内容存放在该目录下, 这些便是我们最终需要交付的 “成品” 了

```txt
public
├── 404.html
├── android-chrome-192x192.png
├── android-chrome-512x512.png
├── apple-touch-icon.png
├── browserconfig.xml
├── css
│   └── style.min.<hash>.css
├── favicon-16x16.png
├── favicon-32x32.png
├── favicon.ico
├── index.html
├── index.xml
├── js
│   └── bundle.min.<hash>.js
├── mstile-150x150.png
├── posts
│   ├── index.html
│   ├── index.xml
│   └── my-first-post
│       └── index.html
├── safari-pinned-tab.svg
├── site.webmanifest
├── sitemap.xml
└── tags
    ...
```

可以看到原来的 markdown 格式文件已经编译为了 html, 原来 `static/` 中的文件也都拷贝了过来.

为了之后让我们的构建能够自动化, 这里, 我们先让 `git` 忽略 `public` 目录

```sh
echo '/public' > .gitignore
```

## 推送至 Github

OK, 一切看上去准备就绪了, 提交我们的更改.

```sh
git add .
git commit -m "It begins."
```

然后使用 `hub` 工具在 github 上创建一个同名的 (即 “blog”) 仓库, 并推送我们的提交

```sh
hub create -d "This is my Blog!"
git push origin master
```

> 第一次使用 `hub` 会提示你输入用户名和密码.

成功后, 可以在浏览器中查看创建的仓库

```sh
hub browse
```

好了, 下面我将介绍如何把我们的站点部署到 github pages 上.

## 创建 Github Pages

[Github pages](https://pages.github.com/) 是 github 推出的免费的静态网站托管服务. 对于个人用户, github 提供两种类型的 pages:

1. *User Pages*: 每个用户可配置一个站点, 通过 `http(s)://<username>.github.io` 访问
2. *Project Pages*: 每个仓库可配置一个站点, 通过 `http(s)://<username>.github.io/<repo>` 访问

这里我们选择 user pages.

创建 user pages 非常简单, 我们只需在 github 上创建一个名为 `<username>.github.io` 仓库即可.
之后, 我们将站点的静态文件更新到该仓库的 `master` 分支.

> User pages 只支持将仓库的 master 分支作为站点的发布源, 这也是为什么我们需要使用两个仓库 (`blog` 和 `<username>.github.io`) 的原因.

另外, 建议在仓库的设置页面中, 启用 `Enforce HTTPS` .

![Github Pages Enforce HTTPS](/img/gh-pages-enforce-https.png)

## 使用 Github Actions 部署 Github Pages

[Github Actions](https://github.com/features/actions) 是 github 推出的免费的 `CI/CD` 工具.
其最大的特点就是拥有海量社区编写的 [Actions](https://github.com/marketplace?type=actions), 我们只需组合这些 Actions, 就能实现日常工作的自动化.

### 整理思路

在编写工作流之前, 首先明确我们的目标. 目前, 我们有两个 git 仓库:

1. `blog`: 存储着 hugo 站点的源文件.
2. `<username>.github.io`: 用于存储最终的发布文件.

我们的工作流将分为 4 步:

1. 检出 `blog` 仓库
2. 安装 hugo
3. 使用 `hugo --minify` 命令构建站点, 最终内容会被生成到 `public/` 下
4. 将 `public/` 中的内容推送到 `<username>.github.io` 仓库, 完成部署.

### 配置 Deploy Key

由于涉及 `<username>.github.io` 仓库的写权限, 我们需要先配置好 “Deploy Key” 以便之后工作流能够正常执行.

生成部署密钥:

```sh
ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f gh-pages -N ""
```

得到两个文件:

1. *gh-pages.pub*: 公钥文件
2. *gh-pages*: 私钥文件

在 `<username>.github.io` 仓库设置页的 “Deploy Keys“ 一栏中, 新建部署密钥, 并填入公钥文件 *gh-pages.pub* 中的内容, **同时, 勾选 “Allow write access”**.

在 `blog` 仓库设置页的 “Secrets” 一栏中, 新建一项, 名为 `ACTIONS_DEPLOY_KEY`, 值为私钥文件 *gh-pages* 中的内容.

### 编写工作流

好了, 回到我们本地的 `blog` 目录, 可以开始编写工作流了.

```sh
mkdir -p .github/workflows
touch .github/workflows/deploy.yml
```

`deploy.yml` 的内容如下:

```yaml
name: deploy
on:
  push:
    branches:
    - master    # 只在 master 分支上执行
jobs:
  build-deploy:
    runs-on: ubuntu-18.04
    steps:
    - uses: actions/checkout@v1
      with:
        submodules: true

    - name: Setup Hugo
      uses: peaceiris/actions-hugo@v2
      with:
        hugo-version: '0.60.1'
        extended: true

    - name: Build
      run: hugo --minify

    - name: Deploy
      uses: peaceiris/actions-gh-pages@v2
      env:
        ACTIONS_DEPLOY_KEY: ${{ secrets.ACTIONS_DEPLOY_KEY }}
        EXTERNAL_REPOSITORY: <username>/<username>.github.io
        PUBLISH_BRANCH: master
        PUBLISH_DIR: ./public
```

每个 workflow, 可由一个或多个 `job` 组成;
每个 `job` 由多个 `step` 组成, `step` 按顺序执行;
`step` 中 `uses` 指令表示所使用的 ”action“, 这里用了 3 个 ”action“:

```sh
git add .github/workflows/deploy.yml
git commit -m ""
git push
```

## 自定义域名

## 接入 Google Analytics

## 参考链接

* [Hugo: Quick Start](https://gohugo.io/getting-started/quick-start/)
* [GitHub Actions for GitHub Pages](https://github.com/marketplace/actions/github-pages-action)
