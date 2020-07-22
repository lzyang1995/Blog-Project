---
title: "从源码构建和运行VS Code"
date: 2020-07-22T13:52:49+08:00
draft: false
tags: ["Electron", "VSCode"]
categories: ["前端开发"]
lightgallery: true
---

VS Code是一个基于Electron的开源的代码编辑器。最近由于项目需要，要分析一下VS Code，因此打算先clone它的源码，然后构建运行起来。可能是因为我的网不好，再加上GFW的原因，这个过程异常艰难。因此在这里记录一下整个的构建过程。

<!--more-->

首先我们按照[官方教程](https://github.com/Microsoft/vscode/wiki/How-to-Contribute#prerequisites)，把各种必须的工具安装好，包括Git, NodeJS, Yarn, Python 2.7, 以及windows-build-tools，然后重启电脑。然后我们fork一下[VSCode的repo](https://github.com/microsoft/vscode)并clone到本地。到这里都还比较顺利，接下来用Yarn安装依赖，问题就来了。

首先，我们需要一个FQ工具。然后使用如下命令设置一下yarn的proxy：

```
yarn config set proxy http://localhost:XXXX
yarn config set https-proxy http://localhost:XXXX
```

之后在vscode目录下执行命令yarn，会一直卡在第四步`building fresh packages`。这里我们需要先设置一下yarn的child-concurrency，把它设置成1：

```
yarn config set child-concurrency 1
```

之后再执行yarn，会发现该命令是卡在了Electron的安装上。我先是尝试把`.yarnrc`文件中的`disturl`改成淘宝的镜像<https://npm.taobao.org/mirrors/electron/>，然而并没有用。从报错信息看，是在目录`node_modules\electron`中执行`node install.js`出的问题。该脚本的内容如下：

```js
#!/usr/bin/env node

const version = require('./package').version

const fs = require('fs')
const os = require('os')
const path = require('path')
const extract = require('extract-zip')
const { downloadArtifact } = require('@electron/get')

if (process.env.ELECTRON_SKIP_BINARY_DOWNLOAD) {
  process.exit(0)
}

const platformPath = getPlatformPath()

if (isInstalled()) {
  process.exit(0)
}

// downloads if not cached
downloadArtifact({
  version,
  artifactName: 'electron',
  force: process.env.force_no_cache === 'true',
  cacheRoot: process.env.electron_config_cache,
  platform: process.env.npm_config_platform || process.platform,
  arch: process.env.npm_config_arch || process.arch
}).then(extractFile).catch(err => {
  console.error(err.stack)
  process.exit(1)
})

function isInstalled () {
  try {
    if (fs.readFileSync(path.join(__dirname, 'dist', 'version'), 'utf-8').replace(/^v/, '') !== version) {
      return false
    }

    if (fs.readFileSync(path.join(__dirname, 'path.txt'), 'utf-8') !== platformPath) {
      return false
    }
  } catch (ignored) {
    return false
  }
  
  const electronPath = process.env.ELECTRON_OVERRIDE_DIST_PATH || path.join(__dirname, 'dist', platformPath)
  
  return fs.existsSync(electronPath)
}

// unzips and makes path.txt point at the correct executable
function extractFile (zipPath) {
  return new Promise((resolve, reject) => {
    extract(zipPath, { dir: path.join(__dirname, 'dist') }, err => {
      if (err) return reject(err)

      fs.writeFile(path.join(__dirname, 'path.txt'), platformPath, err => {
        if (err) return reject(err)

        resolve()
      })
    })
  })
}

function getPlatformPath () {
  const platform = process.env.npm_config_platform || os.platform()

  switch (platform) {
    case 'mas':
    case 'darwin':
      return 'Electron.app/Contents/MacOS/Electron'
    case 'freebsd':
    case 'openbsd':
    case 'linux':
      return 'electron'
    case 'win32':
      return 'electron.exe'
    default:
      throw new Error('Electron builds are not available on platform: ' + platform)
  }
}
```

该脚本的功能应该是通过`downloadArtifact`下载Electron压缩包，然后通过`extractFile`函数解压。如果设置了环境变量`ELECTRON_SKIP_BINARY_DOWNLOAD`，或者目录中已经存在解压后的Electron了，就不进行下载。因此，后面我参考了[这篇文章](https://blog.csdn.net/qq_27005821/article/details/102748201)的做法，手动从淘宝镜像<https://npm.taobao.org/mirrors/electron/9.1.0/>下载`electron-v9.1.0-win32-x64.zip`（我使用的win10 64位系统, vscode的.yarnrc中指定的Electron版本是9.1.0），并放到项目目录的`node_modules\electron`中，重命名为`electron.zip`，并对`install.js`进行如下修改：

```js
#!/usr/bin/env node

const version = require('./package').version

const fs = require('fs')
const os = require('os')
const path = require('path')
const extract = require('extract-zip')
const { downloadArtifact } = require('@electron/get')

if (process.env.ELECTRON_SKIP_BINARY_DOWNLOAD) {
  process.exit(0)
}

const platformPath = getPlatformPath()

if (isInstalled()) {
  process.exit(0)
}

// downloads if not cached
/*downloadArtifact({
  version,
  artifactName: 'electron',
  force: process.env.force_no_cache === 'true',
  cacheRoot: process.env.electron_config_cache,
  platform: process.env.npm_config_platform || process.platform,
  arch: process.env.npm_config_arch || process.arch
}).then(extractFile).catch(err => {
  console.error(err.stack)
  process.exit(1)
})*/
extractFile("./electron.zip").catch(err => {
  console.error(err.stack)
  process.exit(1)
})

function isInstalled () {
  try {
    if (fs.readFileSync(path.join(__dirname, 'dist', 'version'), 'utf-8').replace(/^v/, '') !== version) {
      return false
    }

    if (fs.readFileSync(path.join(__dirname, 'path.txt'), 'utf-8') !== platformPath) {
      return false
    }
  } catch (ignored) {
    return false
  }
  
  const electronPath = process.env.ELECTRON_OVERRIDE_DIST_PATH || path.join(__dirname, 'dist', platformPath)
  
  return fs.existsSync(electronPath)
}

// unzips and makes path.txt point at the correct executable
function extractFile (zipPath) {
  return new Promise((resolve, reject) => {
    extract(zipPath, { dir: path.join(__dirname, 'dist') }, err => {
      if (err) return reject(err)

      fs.writeFile(path.join(__dirname, 'path.txt'), platformPath, err => {
        if (err) return reject(err)

        resolve()
      })
    })
  })
}

function getPlatformPath () {
  const platform = process.env.npm_config_platform || os.platform()

  switch (platform) {
    case 'mas':
    case 'darwin':
      return 'Electron.app/Contents/MacOS/Electron'
    case 'freebsd':
    case 'openbsd':
    case 'linux':
      return 'electron'
    case 'win32':
      return 'electron.exe'
    default:
      throw new Error('Electron builds are not available on platform: ' + platform)
  }
}
```

之后手动执行`node install.js`。然后，我们设置环境变量`ELECTRON_SKIP_BINARY_DOWNLOAD`，防止后面执行yarn时我们下载并解压的Electron被覆盖掉：

```
set ELECTRON_SKIP_BINARY_DOWNLOAD=1
```

然后再在项目根目录下执行yarn。依赖安装就会顺利完成。

后面就是Build以及Run的过程。首先在命令行中执行`yarn watchd`，编译成功之后，打开另一个命令行执行`.\scripts\code.bat`。这里又会下载一遍Electron......又是漫长的等待。主要是这个bat文件会执行`node build/lib/electron.js`，这个js文件会检查项目根目录的`.build\electron`目录中的`version`文件（之前手动下载的Electron压缩包中就有这个`version`文件，里面是Electron的版本号），如果这个文件不存在，或者其中记录的版本号不对，则会去下载Electron。本来我在考虑如果我直接把手动下载的压缩包解压到`.build\electron`目录中行不行，不过在等了半个多小时之后它竟然自己下载完了，所以也就不用折腾了......

下载完Electron并同步内置的Extension之后，VS Code就能启动起来了，如下图所示：

{{< style "text-align: center;" >}}
{{< image src="/images/VSCode-build/success_run.png" caption="成功运行的VS Code" alt="成功运行的VS Code" title="成功运行的VS Code" >}}
{{< /style >}}

## 参考资料

1. <https://github.com/Microsoft/vscode/wiki/How-to-Contribute>
2. <https://stackoverflow.com/questions/51508364/yarn-there-appears-to-be-trouble-with-your-network-connection-retrying>
3. <https://blog.csdn.net/qq_27005821/article/details/102748201>
4. <https://github.com/yarnpkg/yarn/issues/7450>

