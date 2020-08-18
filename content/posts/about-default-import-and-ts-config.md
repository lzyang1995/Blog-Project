---
title: "关于ESM的default import以及TypeScript的坑"
date: 2020-08-18T20:23:36+08:00
draft: false
tags: ["ESM", "TypeScript"]
categories: ["前端开发"]
lightgallery: true
---

最近实习期间在做一个基于Electron的开发调试工具，然后想利用[node-pty](https://github.com/microsoft/node-pty)以及[xterm](https://github.com/xtermjs/xterm.js/)把terminal集成进来，就像VSCode里面的terminal一样，然后遇到了大坑。研究了很久才发现原来是使用默认导入（default import）的问题……

<!--more-->

原本我对于默认、非默认导入，以及不同的模块化的方式（CommonJS, ESM, AMD等）的关系，还有TypeScript的配置文件的认识都不是很深入。直到前几天，我和往常一样，用默认导入的方式引入node-pty：

```js
import os from 'os';
import pty from 'node-pty';

const shell = process.env[os.platform() === 'win32' ? 'COMSPEC' : 'SHELL'];
const ptyProcess = pty.spawn(shell, [], {
  name: 'xterm-color',
  cols: 80,
  rows: 30,
  cwd: process.cwd(),
  env: process.env
});
```

结果报错：

{{< style "text-align: center;" >}}
{{< image src="/images/about-default-import-and-ts-config/spawn_undefined.png" caption="导入node-pty出现错误" alt="导入node-pty出现错误" title="导入node-pty出现错误" >}}
{{< /style >}}

意思就是我导入的pty是undefined。然后我就纳闷了，为什么会引入一个undefined？一开始以为是electron导入native module的问题，因为node-pty是一个navive module，而electron导入native module似乎很容易因为各种原因出问题，比如自己使用的Node版本和Electron中的Node版本不一致。然后以为是Webpack打包的问题，因为对于某些native module而言，打包会破坏原有的目录结构，如果native代码通过相对路径引用了JS代码，就会出现问题(https://github.com/JoshuaWise/better-sqlite3/issues/126#issuecomment-535459620)。所以我也尝试了在Webpack中将node-pty设置为external，然后利用copy-webpack-plugin把node_modules下的node-pty拷到Webpack输出目录的相应位置。然而这些都不是问题的关键。为了知道为什么会出现这个问题，我们需要先了解不同的模块化方式的关系以及TypeScript的相关配置。

## 通过ESM方式引入CommonJS模块

在使用Webpack的时候，我们一般使用ESM语法来引入模块。而被引入的模块一般都是CommonJS语法。此时，ESM的三种引入方式和CommonJS的引入方式存在下面的对应关系：

```js
import * as obj from 'module' <===> const obj = require('module');
import {a, b, c} from 'module' <===> const {a, b, c} = require('module');
import obj from 'module' <===> const obj = require('module').default;
```

可以看到，对于ESM的默认引入方式，会要求被引入的对象存在一个default属性。然而很多CommonJS模块输出的对象中可能都没有这个属性。对于这种情况，Babel会给输出的对象添加一个default属性，使得我们可以使用默认引入的方式。至于TypeScript，它提供了两个相关的配置项：esModuleInterop和allowSyntheticDefaultImports。

## TypeScript配置中的esModuleInterop及allowSyntheticDefaultImports

esModuleInterop也是为了解决ESM的默认引入的问题。在Webpack关于TypeScript的配置教程中(https://webpack.js.org/guides/typescript/)，有这样的描述：

{{< style "text-align: center;" >}}
{{< image src="/images/about-default-import-and-ts-config/webpack-ts.png" caption="Webpack文档中关于这两个选项的描述" alt="Webpack文档中关于这两个选项的描述" title="Webpack文档中关于这两个选项的描述" >}}
{{< /style >}}

当我们把esModuleInterop设置为true时，allowSyntheticDefaultImports也自动设置为true了。然而，设置这两项之后，使用default import就没有问题了吗？当然不是。我的项目出现错误的时候，这两项都已经被设置为true了。那么问题出在哪呢？我们可以先看esModuleInterop的不同设置下，TypeScript编译器输出的JS代码。TypeScript的文档(https://www.typescriptlang.org/tsconfig#esModuleInterop)提供了一个例子，对于下面的TypeScript代码：

```ts
import * as fs from "fs";
import _ from "lodash";

fs.readFileSync("file.txt", "utf8");
_.chunk(["a", "b", "c", "d"], 2);
```

当esModuleInterop为false时，输出如下：

```js
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
const fs = require("fs");
const lodash_1 = require("lodash");
fs.readFileSync("file.txt", "utf8");
lodash_1.default.chunk(["a", "b", "c", "d"], 2);
```

当esModuleInterop为true时，输出如下：

```js
"use strict";
var __createBinding = (this && this.__createBinding) || (Object.create ? (function(o, m, k, k2) {
    if (k2 === undefined) k2 = k;
    Object.defineProperty(o, k2, { enumerable: true, get: function() { return m[k]; } });
}) : (function(o, m, k, k2) {
    if (k2 === undefined) k2 = k;
    o[k2] = m[k];
}));
var __setModuleDefault = (this && this.__setModuleDefault) || (Object.create ? (function(o, v) {
    Object.defineProperty(o, "default", { enumerable: true, value: v });
}) : function(o, v) {
    o["default"] = v;
});
var __importStar = (this && this.__importStar) || function (mod) {
    if (mod && mod.__esModule) return mod;
    var result = {};
    if (mod != null) for (var k in mod) if (k !== "default" && Object.hasOwnProperty.call(mod, k)) __createBinding(result, mod, k);
    __setModuleDefault(result, mod);
    return result;
};
var __importDefault = (this && this.__importDefault) || function (mod) {
    return (mod && mod.__esModule) ? mod : { "default": mod };
};
Object.defineProperty(exports, "__esModule", { value: true });
const fs = __importStar(require("fs"));
const lodash_1 = __importDefault(require("lodash"));
fs.readFileSync("file.txt", "utf8");
lodash_1.default.chunk(["a", "b", "c", "d"], 2);
```

可以看到，当esModuleInterop为false时，就是使用之前提到的方式来引入CommonJS模块。而当esModuleInterop为true时，TypeScript会提供一些helper function来辅助引入模块。

那么为什么使用ESM的默认方式引入node-pty会出问题？我们可以看一下note-pty模块的入口文件(lib/index.js)的代码：

```js
"use strict";
/**
 * Copyright (c) 2012-2015, Christopher Jeffrey, Peter Sunde (MIT License)
 * Copyright (c) 2016, Daniel Imms (MIT License).
 * Copyright (c) 2018, Microsoft Corporation (MIT License).
 */
Object.defineProperty(exports, "__esModule", { value: true });
var terminalCtor;
if (process.platform === 'win32') {
    terminalCtor = require('./windowsTerminal').WindowsTerminal;
}
else {
    terminalCtor = require('./unixTerminal').UnixTerminal;
}
/**
 * Forks a process as a pseudoterminal.
 * @param file The file to launch.
 * @param args The file's arguments as argv (string[]) or in a pre-escaped
 * CommandLine format (string). Note that the CommandLine option is only
 * available on Windows and is expected to be escaped properly.
 * @param options The options of the terminal.
 * @throws When the file passed to spawn with does not exists.
 * @see CommandLineToArgvW https://msdn.microsoft.com/en-us/library/windows/desktop/bb776391(v=vs.85).aspx
 * @see Parsing C++ Comamnd-Line Arguments https://msdn.microsoft.com/en-us/library/17w5ykft.aspx
 * @see GetCommandLine https://msdn.microsoft.com/en-us/library/windows/desktop/ms683156.aspx
 */
function spawn(file, args, opt) {
    return new terminalCtor(file, args, opt);
}
exports.spawn = spawn;
/** @deprecated */
function fork(file, args, opt) {
    return new terminalCtor(file, args, opt);
}
exports.fork = fork;
/** @deprecated */
function createTerminal(file, args, opt) {
    return new terminalCtor(file, args, opt);
}
exports.createTerminal = createTerminal;
function open(options) {
    return terminalCtor.open(options);
}
exports.open = open;
/**
 * Expose the native API when not Windows, note that this is not public API and
 * could be removed at any time.
 */
exports.native = (process.platform !== 'win32' ? require('../build/Release/pty.node') : null);
//# sourceMappingURL=index.js.map
```

可以看到，它通过给exports对象设置了一些属性，但没有default属性。同时，它在最开始给exports对象设置了`__esModule`属性，值为true(似乎TypeScript编译的输出里面都会存在这条语句)。这个就很坑了。首先，如果我们把esModuleInterop设置为false，那么默认引入肯定不行，因为exports对象中没有default属性；其次，如果我们把esModuleInterop设置为true，通过分析前面的例子，我们发现TS会使用`__importDefault`函数来处理require的输出，也就是exports对象。而只要exports对象中存在`__esModule`属性并且为true，该函数就会直接返回exports对象。而后面会通过exports.default来调用exports中的内容，但exports对象中此时还是没有default属性！这就导致了无论我们把esModuleInterop设置为false还是true，都不能通过ESM的默认导入方式来导入node-pty。除非我们把`Object.defineProperty(exports, "__esModule", { value: true });`删掉，然后把esModuleInterop设置为true，此时`__importDefault`会返回`{ "default": mod }`，就不会有问题了。不过相对于改node_modules中的代码，当然最好还是不用默认引入方式，而是用另外两种方式。这里另外两种方式都不会出现问题。

对于allowSyntheticDefaultImports，该属性并不会影响TypeScript编译输出的代码。当该属性为false时，如果我们通过默认引入方式引入一个没有default属性的模块，就会报错。而为true时则不会报错。

## Electron中的app.allowRendererProcessReuse

Electron目前默认不允许renderer process引入native module。而node-pty会引入.node模块，所以在renderer process中引入node-pty还是会报错。我们可以通过把`app.allowRendererProcessReuse`设置为false来解决这个问题，不过Electron不推荐这样做(https://github.com/electron/electron/issues/18397)。我们应该可以在main process中引入node-pty，然后通过ipc传给renderer process。

## 参考资料

1. https://github.com/JoshuaWise/better-sqlite3/issues/126#issuecomment-535459620
2. https://webpack.js.org/guides/typescript/
3. https://www.typescriptlang.org/tsconfig#esModuleInterop
4. https://github.com/electron/electron/issues/18397
