---
title: "React与Java Spring后台联合开发"
date: 2020-06-15T22:54:07+08:00
draft: false
tags: ["React", "Spring"]
categories: ["前端开发"]
lightgallery: true
---

最近自学了React，用Create React App脚手架做了一个小[demo](https://github.com/lzyang1995/Millionare-Hero)，然而Create React App毕竟是用来做单页应用的，可能不太适合做多页面的网站。并且即将去实习的单位是用Java做后台，因此就在想如何在开发过程中把React和Java Spring框架结合起来。这里我找到了一个教程：<https://spring.io/guides/tutorials/react-and-spring-data-rest/>，提供了很大的帮助，但是该教程在构建后台应用方面说的比较详细，在关键的如何在项目中引入前端框架方面却说的不太清楚。研究了一番之后大致清楚了具体做法和部分原理，在这里记录和分享一下。

<!--more-->

教程的后台构建部分就不再描述了，这里重点关注前端部分。这里的关键是在maven项目中引入和使用一个maven插件（plugin），叫做[frontend-maven-plugin](https://github.com/eirslett/frontend-maven-plugin)。教程在`pom.xml`中插入了如下内容以引入该插件：

```xml
<plugin>
	<groupId>com.github.eirslett</groupId>
	<artifactId>frontend-maven-plugin</artifactId>
</plugin>
```

然而这样是远远不够的。为了能够顺利地使用该插件，我们需要先知道maven的大致工作原理。

maven在我看来是一个Java的项目构建和包管理工具，类似于JavaScript的npm。我们可以使用它来创建项目，以及完成编译、测试、打包等项目生命周期中的各类操作。它把项目的生命周期（lifecycle）定义为了按顺序排列的各个阶段（phase），包括验证（validate）、编译（compile）、测试（test）、打包（package）等等，具体列表可以参考[关于生命周期的文档](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#lifecycle-reference)。我们可以通过命令mvn加上生命周期中的任意阶段来执行**从初始阶段到该阶段的所有阶段的操作**。例如，如果我们执行`mvn package`，那么maven就会执行从验证到打包的所有操作，最终生成JAR（或者WAR、EAR，由项目配置决定）文件。

在maven项目中，项目配置文件叫做`pom.xml`，里面定义了项目相关的所有内容。其中一项就是项目所使用的插件，定义在`<plugins>`标签中。每个插件又包含多种功能，每个功能被称为**goal**，我们可以只执行插件中的某个或某几个goal。**在生命周期的各个阶段，执行的实际上是各个插件的goals**。其中在项目初始化时就已经包含了很多maven自带的插件的goals（可以在项目初始化时的`pom.xml`中看到，定义了很多插件）。对于我们自己加入的插件的goal，也可以在`pom.xml`中指定它在某个阶段运行。这样，当我们每次执行mvn命令时，只要覆盖了该阶段，这个插件的这个goal就会执行。

除此之外，我们也可以在命令行中单独执行插件的goal。命令的格式如下：

```bash
mvn <plugin-name>:<goal> -D<parameter-name>=<value>
```

其中`<plugin-name>`感觉一般就是`pom.xml`中插件的`<artifactId>`的前半部分，例如`<artifactId>`是`spring-boot-maven-plugin`那么`<plugin-name>`就是`spring-boot`。

明白了以上内容，我们就知道应该如何让前面的教程项目工作起来了。我们在`pom.xml`的frontend-maven-plugin部分加入如下内容：

```xml
<plugin>
    <groupId>com.github.eirslett</groupId>
    <artifactId>frontend-maven-plugin</artifactId>
    <version>1.10.0</version>
    <executions>
        <execution>
            <id>install node and npm</id>
            <goals>
                <goal>install-node-and-npm</goal>
            </goals>
            <configuration>
                <!-- See https://nodejs.org/en/download/ for latest node and npm (lts) versions -->
                <nodeVersion>v12.18.0</nodeVersion>
            </configuration>
        </execution>

        <execution>
            <id>npm install</id>
            <goals>
                <goal>npm</goal>
            </goals>
        </execution>

        <execution>
            <id>webpack build</id>
            <goals>
                <goal>webpack</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

上面的xml中给这个插件定义了三个execution，每个execution对应着一个goal和一个生命周期的阶段（phase），这里我们使用这些goal的默认阶段（在该插件中定义为`generate-resources`），那么这三个goal就会在这个阶段自动执行。这里对于第一个goal我们加入了`<configuration>`标签进行配置，指定要安装的Node的版本。由于该版本自带npm，所以这里不用再指定npm的版本了。

接下来，我们只需要在命令行中执行spring-boot的命令：

```bash
mvnw spring-boot:run
```

这里的`mvnw`是maven wrapper，对应着该项目专属的maven而不是系统的全局maven（详见[此文章](https://www.liaoxuefeng.com/wiki/1252599548343744/1305148057976866)）。那么我们就可以看到，前面在`pom.xml`中定义的三个goal全部都执行了：

{{< style "text-align: center;" >}}
{{< image src="/images/React-Spring/spring-boot-run.png" caption="执行命令mvnw spring-boot:run" alt="执行命令mvnw spring-boot:run" title="执行命令mvnw spring-boot:run" >}}
{{< /style >}}

这里其实还有一个问题，就是我们实际运行的是插件spring-boot的一个goal，叫做run，为什么会把`generate-resources`阶段的三个`frontend-maven-plugin`的goal也执行了呢？我感觉原因是因为spring-boot的这个goal实际上就是运行到了生命周期的`test-compile`阶段：

{{< style "text-align: center;" >}}
{{< image src="/images/React-Spring/test-compile.png" caption="输出中的test-compile" alt="输出中的test-compile" title="输出中的test-compile" >}}
{{< /style >}}

而该阶段在`generate-resources`之后，所以这三个goal也会执行。

现在前端模块已经打包并生成完毕了，如下图所示：

{{< style "text-align: center;" >}}
{{< image src="/images/React-Spring/built.png" caption="webpack生成的文件" alt="webpack生成的文件" title="webpack生成的文件" >}}
{{< /style >}}

目录`src/main/resources`中的内容都会被打包。所以我们也可以在`target`目录中看到他们。现在我们可以正常访问网站根目录（index.html）了：

{{< style "text-align: center;" >}}
{{< image src="/images/React-Spring/webpage.png" caption="index.html页面" alt="index.html页面" title="index.html页面" >}}
{{< /style >}}

当然，我们也可以不把`frontend-maven-plugin`的这三个goal放在`pom.xml`里面，而是在命令行中执行：

{{< style "text-align: center;" >}}
{{< image src="/images/React-Spring/exec-in-cmd.png" caption="命令行执行前端插件" alt="命令行执行前端插件" title="命令行执行前端插件" >}}
{{< /style >}}

不过这样我们每次修改前端代码之后都要在命令行中输入这些命令来编译前端内容，显然没有只执行一条命令`mvnw spring-boot:run`方便。

当然，该教程的例子中仍然只有一个网页。对于多个网页，我们可以给每个网页都在webpack中设置一个入口js文件（见<https://webpack.js.org/concepts/entry-points/#multi-page-application>），然后把生成的各个js文件分别插入到各个页面中即可。

## 参考资料

1. <https://spring.io/guides/tutorials/react-and-spring-data-rest/>
2. <https://github.com/eirslett/frontend-maven-plugin>
3. <https://maven.apache.org/guides/getting-started/index.html>
4. <https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html>
5. <https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html>
6. <https://maven.apache.org/guides/mini/guide-configuring-plugins.html>
7. <https://www.liaoxuefeng.com/wiki/1252599548343744/1305148057976866>
8. <https://webpack.js.org/concepts/entry-points/#multi-page-application>

