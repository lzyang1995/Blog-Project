---
title: "Webpack中的publicPath选项"
date: 2020-09-23T21:52:21+08:00
draft: false
tags: ["Webpack"]
categories: ["前端开发"]
lightgallery: true
---

在Webpack配置中有一个output.publicPath选项，可以用来给资源的URL加上前缀。除了这个设置以外，很多loader以及plugin也可以设置publicPath。这里我们探索一下output.publicPath以及两个比较常用的loader：file-loader和MiniCssExtractPlugin.loader的publicPath选项的效果。实验时的Webpack、file-loader以及MiniCssExtractPlugin的版本分别为5.0.0-beta.30，6.1.0和0.11.2。

首先，初始的Webpack配置定义如下，其中还没有设置任何publicPath：

```js
// Author: Zhiyang Li
// Date: 2020.09.23

const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
    mode: 'development',
    entry: {
        index: './src/index.js',
    },
    devtool: 'source-map',
    output: {
        filename: '[name].bundle.js',
        path: path.resolve(__dirname, 'dist'),
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [MiniCssExtractPlugin.loader, 'css-loader'],
            },
            {
                test: /\.(jpg|png)$/,
                loader: 'file-loader',
                options: {
                    name: '[name].[contenthash].[ext]',
                }
            }
        ],
    },
    plugins: [
        new CleanWebpackPlugin({
            cleanStaleWebpackAssets: false,
        }),
        new HtmlWebpackPlugin({
            title: "Public Path",
        }),
        new MiniCssExtractPlugin({
            filename: '[name].bundle.css',
        }),
    ]
};
```

我们在项目中包含两张图片：icon.png和test.jpg。其中test.jpg在style.css中引入，作为background-image：

style.css:

```css
/*
* Author: Zhiyang Li
* Date: 2020.09.23
*/
body {
  background-image: url('./img/test.jpg');
}
```

然后我们的入口文件index.js定义如下：

index.js:

```js
// Author: Zhiyang Li
// Date: 2020.09.23

import './style.css';
import Icon from './img/icon.png';

const img = document.createElement("img");
img.src = Icon;
document.body.append(img);
```

在这种情况下，Webpack的输出如下图所示。此时所有资源的URL前面并没有任何前缀。

{{< style "text-align: center;" >}}
{{< image src="/images/webpack-publicPath/init-output.png" caption="初始配置下的输出" alt="初始配置下的输出" title="初始配置下的输出" >}}
{{< /style >}}

{{< style "text-align: center;" >}}
{{< image src="/images/webpack-publicPath/init-urls.png" caption="初始配置下各资源的URL" alt="初始配置下各资源的URL" title="初始配置下各资源的URL" >}}
{{< /style >}}

## 设置output.publicPath

现在我们将Webpack的output.publicPath设置为"abcde."。重新构建之后，首先生成的HTML文件中css和js的URL前面被加上了"abcde./"：

{{< style "text-align: center;" >}}
{{< image src="/images/webpack-publicPath/output.publicPath-html.png" caption="设置output.publicPath后的HTML文件" alt="设置output.publicPath后的HTML文件" title="设置output.publicPath后的HTML文件" >}}
{{< /style >}}

所以此时打开这个HTML文件会报错。我们手动去掉css和js的URL的前缀之后，再次打开该文档，会发现两张图片的URL也被加上了前缀，不过**前缀中并没有加上"/"**：

{{< style "text-align: center;" >}}
{{< image src="/images/webpack-publicPath/output.publicPath-imgs.png" caption="设置output.publicPath后的图片URL" alt="设置output.publicPath后的图片URL" title="设置output.publicPath后的图片URL" >}}
{{< /style >}}

## 设置file-loader的publicPath

现在我们将Webpack的output.publicPath去掉，然后将file-loader的publicPath选项设置为"abcde."。重新构建之后，生成的HTML文档中的css和js文件的URL不再带有前缀，而两张图片的URL都会加上前缀"abcde./"（**注意这里加上了"/"，而前一种情况下是没有的**）：

{{< style "text-align: center;" >}}
{{< image src="/images/webpack-publicPath/fileloader.publicPath-imgs.png" caption="设置file-loader的publicPath后的图片URL" alt="设置file-loader的publicPath后的图片URL" title="设置file-loader的publicPath后的图片URL" >}}
{{< /style >}}

这个结果也很合理，因为两张图片都是file-loader处理的。

## 设置MiniCssExtractPlugin.loader的publicPath

现在我们去掉前面设置的其他publicPath，只将MiniCssExtractPlugin.loader的publicPath设置为"abcde."。重新构建之后，只有css中引用的图片URL前面被加上了"abcde./"：

{{< style "text-align: center;" >}}
{{< image src="/images/webpack-publicPath/csspluginloader.publicPath-imgs.png" caption="设置MiniCssExtractPlugin.loader的publicPath后的图片URL" alt="设置MiniCssExtractPlugin.loader的publicPath后的图片URL" title="设置MiniCssExtractPlugin.loader的publicPath后的图片URL" >}}
{{< /style >}}

## 运行时设置publicPath

在Webpack的文档中提到了一种[运行时设置publicPath的方法](https://webpack.js.org/guides/public-path/#on-the-fly)。我也尝试了一下在运行时将其设置为"abcde."。结果比较奇怪，只有img标签的图片的URL前面被加上了"abcde."（**没有"/"**）：

{{< style "text-align: center;" >}}
{{< image src="/images/webpack-publicPath/runtime.publicPath-imgs.png" caption="运行时设置publicPath的情况" alt="运行时设置publicPath的情况" title="运行时设置publicPath的情况" >}}
{{< /style >}}

## 修改file-loader和MiniCssExtractPlugin的文件输出位置

这里我们顺便提一下file-loader和MiniCssExtractPlugin的文件输出位置的修改方法。file-loader可以通过选项outputPath来改变文件的输出位置，默认情况下是输出在Webpack的输出根目录，在我们的配置下是"./dist"。如果我们将file-loader的outputPath设置为"imgs"，那么文件会被输出到"./dist/imgs"中，即该设置是相对于Webpack的输出根目录。与此同时，设置outputPath之后，file-loader会自动给每个文件的URL前面加上相应的前缀，因此我们就不用再设置publicPath了，如下图所示：

{{< style "text-align: center;" >}}
{{< image src="/images/webpack-publicPath/fileloader-outputPath.png" caption="设置file-loader的outputPath后的图片URL" alt="设置file-loader的outputPath后的图片URL" title="设置file-loader的outputPath后的图片URL" >}}
{{< /style >}}

而对于MiniCssExtractPlugin，我们可以通过其filename选项来修改css文件输出位置。这个设置也是相对于Webpack的输出根目录。例如将其设置为"css/\[name\].bundle.css"，则css文件会被输出到"./dist/css"中。此时css中引用的图片的URL会出错，需要设置相应的publicPath来解决。

## 参考资料

1. <https://webpack.js.org/guides/public-path/>
2. <https://webpack.js.org/plugins/mini-css-extract-plugin/>
3. <https://webpack.js.org/loaders/file-loader/>
