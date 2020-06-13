---
title: "使用Hugo+GitHub Pages构建博客网站"
date: 2020-05-09T20:22:16+08:00
draft: false
featuredImage: "/images/Hugo-Blog/hugo-logo-wide.png"
tags: ["Hugo", "GitHub Pages"]
categories: ["博客搭建"]
lightgallery: true
---

[Hugo](https://gohugo.io/)是一个基于Go语言的静态网站构建工具，它提供了大量可供选择的模板主题（Theme），帮助我们更加快捷地搭建自己的网站，并且可以非常方便地部署在GitHub Pages上。

<!--more-->

下面分享一下我利用Hugo和Github Pages搭建博客网站（也就是这个网站）的一些经验。

## Hugo安装

Hugo支持各种主流操作系统。对于Windows系统，我们可以直接使用预先编译好的可执行文件（<https://github.com/gohugoio/hugo/releases>）。Windows下的可执行文件有两个版本，一个是常规版本，另一个是扩展版本（extended），这两个版本的区别在于后者支持Sass，而前者不支持，所以如果后面选择的模板主题中使用了Sass，就必须下载扩展版本。

下载完成之后，我们可以把Hugo可执行文件添加到环境变量Path中，方便在命令行中调用。

## 网站构建

首先在命令行中执行`hugo new site personal_blog`，这会在当前目录下新建一个名为`personal_blog`的目录，并在该目录下生成一些初始文件及目录：

{{< style "text-align: center;" >}}
{{< image src="/images/Hugo-Blog/initialize.png" caption="初始文件及目录" alt="初始文件及目录" title="初始文件及目录" >}}
{{< /style >}}

其中一些文件及目录的作用如下：

+ **config.toml**: 网站的配置文件，可以用于设置网站的标题、语言、使用的模板主题、作者信息等等，具体选项可以参考各个模板主题的文档
+ **themes**: 用于存放模板主题相关的文件
+ **static**: 可用于存放网站用到的图像、视频、音乐等资源
+ **content**: 用于存放Markdown文件，如撰写的博客文档

接下来我们可以前往<https://themes.gohugo.io/>选择一个喜欢的模板主题。里面有非常非常多的主题可供选择：

{{< style "text-align: center;" >}}
{{< image src="/images/Hugo-Blog/themes.png" caption="主题" alt="主题" title="主题" >}}
{{< /style >}}

这里以主题[LoveIt](https://github.com/dillonzq/LoveIt)为例进行说明。首先我们进入`personal_blog`目录，然后在命令行中执行以下命令将该主题下载到`themes`目录中：

```
git init
git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt
```

然后在配置文件中加入如下内容，指定使用的主题和版本（注意不要删去原有内容）：

```toml
theme = "LoveIt"

[params]
  version = "0.2.X"
```

这样基本的网站框架就搭建好了。我们可以在本地启动服务查看网站的效果，方法是在命令行中输入`hugo server`，然后访问网址<localhost:1313/>。大致的效果如下：

{{< style "text-align: center;" >}}
{{< image src="/images/Hugo-Blog/structure.png" caption="网站框架" alt="网站框架" title="网站框架" >}}
{{< /style >}}

## 网站配置

刚才我们看到的网站非常简陋，因此接下来我们需要对网站进行配置以丰富网站的内容，方法是对配置文件**config.toml**进行修改。不同的模板主题支持的参数可能不太一样，我们选择的主题LoveIt所支持的各种参数和功能可以参考其[文档](https://hugoloveit.com/zh-cn/theme-documentation-basics/)。下面是一个比较简单的配置文件的例子，主要设置了语言、标题、主题、主题版本、搜索、菜单栏、作者等。

```toml
baseURL = "http://example.org/"
# [en, zh-cn, fr, ...] 设置默认的语言
defaultContentLanguage = "zh-cn"
# 网站语言, 仅在这里 CN 大写
languageCode = "zh-CN"
# 是否包括中日韩文字
hasCJKLanguage = true
# 网站标题
title = "志洋的博客"

# 更改使用 Hugo 构建网站时使用的默认主题
theme = "LoveIt"

[params]
  # LoveIt 主题版本
  version = "0.2.X"
  defaultTheme = "dark"
  [params.search]
    enable = true
    # type of search engine ("lunr", "algolia")
    type = "lunr"
    # max index length of the chunked content
    contentLength = 4000
    # placeholder of the search bar
    placeholder = ""
    # LoveIt NEW | 0.2.1 max number of results length
    maxResultLength = 10
    # LoveIt NEW | 0.2.3 snippet length of the result
    snippetLength = 30
    # LoveIt NEW | 0.2.1 HTML tag name of the highlight part in results
    highlightTag = "em"
    # LoveIt NEW | 0.2.4 whether to use the absolute URL based on the baseURL in search index
    absoluteURL = false
    [params.search.algolia]
      index = ""
      appID = ""
      searchKey = ""

[menu]
  [[menu.main]]
    identifier = "posts"
    # 你可以在名称 (允许 HTML 格式) 之前添加其他信息, 例如图标
    pre = ""
    # 你可以在名称 (允许 HTML 格式) 之后添加其他信息, 例如图标
    post = ""
    name = "文章"
    url = "/posts/"
    # 当你将鼠标悬停在此菜单链接上时, 将显示的标题
    title = ""
    weight = 1
  [[menu.main]]
    identifier = "tags"
    pre = ""
    post = ""
    name = "标签"
    url = "/tags/"
    title = ""
    weight = 2
  [[menu.main]]
    identifier = "categories"
    pre = ""
    post = ""
    name = "分类"
    url = "/categories/"
    title = ""
    weight = 3
  [[menu.main]]
    identifier = "github"
    pre = "<i class='fab fa-github fa-fw'></i>"
    post = ""
    name = ""
    url = "https://github.com/lzyang1995"
    title = "GitHub"
    weight = 4

[outputs]
  home = ["HTML", "RSS", "JSON"]

[author]
  name = "李志洋"
```

设置修改完成后，网站的大致效果如下：

{{< style "text-align: center;" >}}
{{< image src="/images/Hugo-Blog/after_config.png" caption="修改设置后的效果" alt="修改设置后的效果" title="修改设置后的效果" >}}
{{< /style >}}

## 撰写博客

我们可以通过`hugo new`命令来新建一篇博文，如`hugo new posts/my-first-post.md`，这样会在目录`content/posts`下新建一个Markdown文件`my-first-post.md`，其初始内容如下：

```markdown
---
title: "My First Post"
date: 2020-05-09T22:51:33+08:00
draft: true
---


```

可以看到其中初始化了几个前置参数（Front Matter）。我们在后面加入一些内容，如`This is my first post`，然后重新启动服务：`hugo server -D`。需要注意的是这里需要加上选项`-D`，表示将Draft博文也显示在网站中。否则，新建的博文将不会显示，因为新建的博文的前置参数中draft默认为true。或者我们也可以将该draft参数修改为false。此时访问<localhost:1313/>，我们可以看到自己的博客已经显示出来：

{{< style "text-align: center;" >}}
{{< image src="/images/Hugo-Blog/first_post.png" caption="加入一篇博文" alt="加入一篇博文" title="加入一篇博文" >}}
{{< /style >}}

## 将网站部署到GitHub Pages上

首先，我们新建一个GitHub仓库`blog-repo`，用于存放前文所创建的所有网站文件。我们将其clone到本地，假设目录名也叫`blog-repo`，然后将前文中`personal_blog`目录下的所有文件和目录（.git除外）复制到`blog-repo`目录下。接下来，我们再新建一个GitHub仓库`<username>.github.io`，用于存放Hugo最终生成的网站。这里的`<username>`是你的GitHub用户名。然后，我们在`blog-repo`目录下执行以下命令：

```
git submodule add -b master https://github.com/<username>/<username>.github.io.git public
```

上述命令会将`<username>.github.io`仓库下载到`blog-repo/public`目录下。这个目录也是Hugo生成的网站默认所在的目录，所以后续生成网站之后，我们可以直接将`blog-repo/public`目录下的内容push到远端仓库中，这样就完成了网站在GitHub Pages上的部署。我们可以将部署过程写成一个shell script（参考自Hugo官方教程）:

```
#!/bin/sh

# If a command fails then the deploy stops
set -e

printf "\033[0;32mDeploying updates to GitHub...\033[0m\n"

# Build the project.
hugo # if using a theme, replace with `hugo -t <YOURTHEME>`

# Go To Public folder
cd public

# Add changes to git.
git add .

# Commit changes.
msg="rebuilding site $(date)"
if [ -n "$*" ]; then
	msg="$*"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master
```

这样每当我们撰写了新的博文，或者修改了网站设置，就可以执行上面的脚本将网站重新部署到GitHub Pages上。在Windows环境下，我们可以通过Git Bash或者Windows Subsystem for Linux来执行shell script。

## 参考资料
1. <https://gohugo.io/getting-started/quick-start/>
2. <https://gohugo.io/hosting-and-deployment/hosting-on-github/>
3. <https://hugoloveit.com/zh-cn/theme-documentation-basics/>
4. <https://hugoloveit.com/zh-cn/theme-documentation-content/>
5. <https://mjsmithdev.com/setting-up-hugo-on-windows/>
