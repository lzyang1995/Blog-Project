---
title: "使用Git将本地项目上传到远程仓库"
date: 2020-06-12T21:31:08+08:00
draft: false
tags: ["Git"]
categories: ["GitHub"]
lightgallery: true
---

最近自学了一下React，Redux以及React Router的相关内容，想找个项目练练手，于是找到了B站上的一个教程（<https://www.bilibili.com/video/BV1T7411W72T>），这个教程最后有一个仿百万英雄的答题App的小项目，然后就按照它的功能自己实现了一遍。今天想着把这个项目上传到GitHub上，顺便学习一下自己还不太熟悉的Git。这里记录一下自己的上传过程。

<!--more-->

首先进入我们的项目目录，使用如下命令初始化本地仓库：

```bash
git init
```

该命令会在项目目录中建立一个`.git`目录。不过实际上在Create React App创建的项目目录中已经存在`.git`目录了。

此时项目目录中的所有文件都处于untracked状态。接下来使用如下命令让Git来track目录中的所有文件：

```bash
git add *
```

这个命令同时会stage所有的文件，让它们处于可以commit的状态。注意`.gitignore`文件中指定的文件不会被track。

{{< admonition type=tip open=true >}}
当我们发现自己通过add命令加入了一个不应该加入的文件时，此时更新`.gitignore`文件是没有用的，因为`.gitignore`只能让untracked文件保持
untracked，并不能把tracked文件变回untracked。假设我们想移除的文件是`a.txt`，那么我们需要执行如下命令：

```bash
git rm --cached a.txt
```

这里的`--cached`选项是为了让`a.txt`仍然存在于项目目录中，只是变成untracked文件，否则`a.txt`会被删除。之后我们需要执行commit命令来将
这个改变记录在快照（snapshot）中。
{{< /admonition >}}

接下来就可以使用commit命令保存当前的快照（snapshot）：

```bash
git commit -m "message"
```

现在我们还没有指定要把这个项目上传到哪个远程仓库。这里我们登录GitHub新建一个仓库，如下图所示，URL为<https://github.com/lzyang1995/Millionare-Hero.git>:

{{< style "text-align: center;" >}}
{{< image src="/images/Git-Upload/create-repo_marked.png" caption="创建仓库" alt="创建仓库" title="创建仓库" >}}
{{< /style >}}

这里由于我已经创建过这个仓库了，所以图中显示的是已经存在。这里我选择初始化时加入一个README和Licence，实际上什么都不加的话后续会方便一点。

现在我们就可以在本地添加对应的远程仓库的信息了。执行如下命令来添加远程仓库：

```bash
git remote add origin https://github.com/lzyang1995/Millionare-Hero.git
```

这里我们把这个远程仓库命名为了origin。接下来我们执行fetch命令来获取远程仓库的branch信息：

```bash
git fetch origin
```

然后我们可以设置一下本地的master branch的upstream branch，把它设置成远程仓库的master branch，即origin/master:

```
git branch -u origin/master
```

我们可以通过`git branch -vv`命令来查看upstream branch的设置情况。接下来我们把origin/master来merge进我们本地的master branch：

```bash
git merge origin/master --allow-unrelated-histories
```

注意这个地方我们必须加上`--allow-unrelated-histories`，因为本地的master branch和origin/master没有共同的父节点，默认是不允许merge的。

这里在merge的时候出现了一个小问题，就是我远程新建仓库是初始化了一个README，结果本地项目目录中也有一个README（Create React App生成的），所以出现了merge conflict。解决方法就是自行修改README，保存后执行add和commit命令：

```bash
git add README.md
git commit -m "Merge the projects"
```

这样merge conflict就解决了。现在我们可以把我们的项目上传到远程仓库了，执行如下命令：

```bash
git push origin master
```

这个命令是`git push origin master:master`的缩写，作用是把本地的master branch上传到origin/master branch。

到这里，项目就已经上传完成了。不过，后续我把项目clone到本地，执行`npm install`，然后执行`npm start`试着运行的时候，会报编译错误，说无法找到某某模块，而这些模块实际上在原项目中已经安装过了。后来发现问题可能出在`cnpm`上，原项目的这些模块是通过`cnpm`安装的，我后面又在原项目中用`npm`把这些模块重新安装了一遍，再次push到远程仓库，然后clone到本地试运行，就没有再报错了。

## 参考资料

1. <https://git-scm.com/book/en/v2>

