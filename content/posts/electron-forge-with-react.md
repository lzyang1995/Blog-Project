---
title: "搭建基于Webpack, TypeScript和React的Electron开发框架"
date: 2020-07-29T10:33:33+08:00
draft: false
tags: ["React", "Electron"]
categories: ["前端开发"]
lightgallery: true
---

最近的实习项目需要基于Electron来做，如果能把Webpack, TypeScript和React也融入到开发过程中就再好不过了。这里记录一下搭建这样一个开发框架的过程。

<!--more-->

搭建过程其实并不复杂，我是选择基于[Electron Forge](https://www.electronforge.io/)来做。Electron Forge已经提供了一个包含Webpack和TypeScript的模板（<https://www.electronforge.io/templates/typescript-+-webpack-template>），我们只需要再向里面加入React即可。

首先，使用如下命令初始化Electron Forge模板：

```
npx create-electron-app my-new-app --template=typescript-webpack
```

在生成的项目目录中，`src`目录对应着源文件，其中`index.ts`是Electron main process的代码，在Webpack打包之后对应于`.webpack\main\index.js`。`index.html`是Electron renderer process加载的HTML文件，对应于`.webpack\renderer\main_window\index.html`。`renderer.ts`对应于`.webpack\renderer\main_window\index.js`，在Webpack打包之后会被`index.html`通过script标签引入。在项目目录中执行`npm start`就可以启动Electron程序。

接下来，我们需要安装React相关的依赖：

```
npm install --save react react-dom
npm install --save-dev @types/react @types/react-dom
```

为了能够使用JSX，我们需要在`tsconfig.json`的`compilerOptions`中加入`"jsx": "react"`。同时，使用JSX的源文件拓展名必须是`.tsx`而不能是`.ts`。

到这里开发的框架就已经搭建完成了。我们可以测试一下React的使用。首先，修改`src\index.html`，加入一个div元素：

{{< style "text-align: center;" >}}
{{< image src="/images/electron-forge-with-react/index.html.png" caption="index.html" alt="index.html" title="index.html" >}}
{{< /style >}}

然后将`renderer.ts`重命名为`renderer.tsx`，并修改如下：

{{< style "text-align: center;" >}}
{{< image src="/images/electron-forge-with-react/renderer.tsx.png" caption="renderer.tsx" alt="renderer.tsx" title="renderer.tsx" >}}
{{< /style >}}

由于我们重命名了renderer文件，我们需要修改`package.json`中的相关配置：

{{< style "text-align: center;" >}}
{{< image src="/images/electron-forge-with-react/package.json.png" caption="package.json" alt="package.json" title="package.json" >}}
{{< /style >}}

然后，执行`npm start`命令，可以看到如下结果：

{{< style "text-align: center;" >}}
{{< image src="/images/electron-forge-with-react/Electron程序界面.png" caption="Electron程序界面" alt="Electron程序界面" title="Electron程序界面" >}}
{{< /style >}}

## 参考资料

1. <https://www.typescriptlang.org/docs/handbook/react-&-webpack.html>
2. <https://www.typescriptlang.org/docs/handbook/jsx.html#basic-usage>
3. <https://ankitbko.github.io/2019/08/electron-forge-with-react-and-typescript/>
4. <https://www.electronforge.io/templates/typescript-+-webpack-template>
