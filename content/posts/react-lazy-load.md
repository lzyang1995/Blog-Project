---
title: "React中的懒加载"
date: 2020-09-22T17:05:53+08:00
draft: false
tags: ["React", "Lazy Load"]
categories: ["前端开发"]
lightgallery: true
---

React中的懒加载（Lazy Load）有两种实现方式，一个是通过`React.lazy`，另一个是通过叫做`loadable components`（<https://loadable-components.com/>）的包。这两种方式用法和效果几乎是一样的，并且它们的懒加载都是基于Webpack的动态加载（dynamic import）实现的。当我们在代码中动态加载一个模块，并且用Webpack打包，那么该模块会单独生成一个chunk，并且默认是懒加载的模式，即要用到该模块时才会加载这个chunk。

React中的这两种懒加载的方式是用于懒加载组件（Component）。在自己试了几个例子之后，个人感觉**懒加载的组件是在首次渲染时加载**（当然如果使用prefetch或者preload的话，一般会加载的更早一些）。

## React.lazy

`React.lazy`是React官方实现的懒加载方式，一般会和`Suspense`以及`Error Boundary`一起使用。`Suspense`用来提供组件加载时的fallback UI，`Error Boundary`用来提供加载出错（如网络错误）时的fallback UI。在[官方文档](https://reactjs.org/docs/code-splitting.html#reactlazy)中给出了一个简单的懒加载的例子，如下所示：

```jsx
import React, { Suspense } from 'react';

const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```

不过如果我们直接渲染这个`MyComponent`组件的话，其实看不到懒加载的效果，因为这个组件立即就渲染了`OtherComponent`，所以`OtherComponent`立刻就被加载了。

为了测试懒加载的效果，我自己实现了一个App组件，代码如下：

```jsx
// Author: Zhiyang Li
// Date: 2020.09.22

import React, { Suspense } from 'react';

class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      currentComponent: null,
    };
  }

  getComponent() {
    const { currentComponent } = this.state;

    if (currentComponent === null) {
      return null;
    } else if (currentComponent === "OtherComponent") {
      return React.lazy(() => import(/* webpackChunkName: "OtherComponent" */ "./OtherComponent"));
    } else if (currentComponent === "AnotherComponent") {
      return React.lazy(() => import(/* webpackChunkName: "AnotherComponent" */ "./AnotherComponent"));
    }
  }

  render() {
    return (
      <div>
        <div><Component show={() => this.getComponent()} /></div>
        <button onClick={() => this.setState({ currentComponent: "OtherComponent" })}>
          Show OtherComponent
        </button>
        <button onClick={() => this.setState({ currentComponent: "AnotherComponent" })}>
          Show AnotherComponent
        </button>
        <button onClick={() => this.setState({ currentComponent: null })}>
          Clear
        </button>
      </div>
    );
  }
}

function Component(props) {
  let ComponentToShow = props.show();

  if (ComponentToShow === null) {
    return null;
  } else {
    return (
      <Suspense fallback={<div>Loading...</div>}>
        <ComponentToShow />
      </Suspense>
    );
  }
}

export default App;
```

这个App组件一开始并没有渲染`OtherComponent`以及`AnotherComponent`，只有在按下相应的按钮之后才会渲染。打开测试网页并观察Chrome DevTools的Network部分，可以看到懒加载的效果，如下图所示：

{{< style "text-align: center;" >}}
{{< image src="/images/react-lazy-load/react-lazy.png" caption="React.lazy示例一效果" alt="React.lazy示例一效果" title="React.lazy示例一效果" >}}
{{< /style >}}

我们再看另一个例子。这里我们实现两个模块，第一个模块中的组件`App`通过`React.lazy`加载`OtherComponent`和`AnotherComponent`并渲染，第二个模块中的组件`ShowApp`则会显示一个按钮，按下去会渲染`App`组件（一开始没有渲染）。两个模块的代码分别如下：

AppReactLazyDefault.js:

```jsx
import React, { Suspense } from 'react';

const OtherComponent = React.lazy(() => import(/* webpackChunkName: "OtherComponent" */ './OtherComponent'));
const AnotherComponent = React.lazy(() => import(/* webpackChunkName: "AnotherComponent" */ './AnotherComponent'));

function App() {
    return (
        <div>
            <Suspense fallback={<div>Loading...</div>}>
                <section>
                    <OtherComponent />
                    <AnotherComponent />
                </section>
            </Suspense>
        </div>
    );
}

export default App;
```

ShowAppReactLazyDefault.js:

```jsx
// Author: Zhiyang Li
// Date: 2020.09.22

import React from 'react';
import App from './AppReactLazyDefault';

class ShowApp extends React.Component {
    constructor(props) {
        super(props);

        this.state = {
            show: false,
        };
    }

    render() {
        return (
            <div>
                <div>{ this.state.show ? <App /> : null }</div>
                <button onClick={() => this.setState(state => ({ show: !state.show }))}>
                    toggle
                </button>
            </div>
        );
    }
}

export default ShowApp;
```

我们在页面中渲染`ShowApp`组件，可以发现`OtherComponent`和`AnotherComponent`是在按下按钮之后才加载的，如下图所示：

{{< style "text-align: center;" >}}
{{< image src="/images/react-lazy-load/react-lazy-2.png" caption="React.lazy示例二效果" alt="React.lazy示例二效果" title="React.lazy示例二效果" >}}
{{< /style >}}

这说明懒加载不是发生在`React.lazy`调用的时候，而是发生在组件首次渲染的时候。

## Loadable Components

`Loadable Components`的用法和效果基本上和`React.lazy`一模一样。不过Loadable Components的适用范围更广一些，官方文档中有这样一个和`React.lazy`的[比较](https://loadable-components.com/docs/loadable-vs-react-lazy/#comparison-table)：

{{< style "text-align: center;" >}}
{{< image src="/images/react-lazy-load/comparison.png" caption="Loadable Components和React.lazy的比较" alt="Loadable Components和React.lazy的比较" title="Loadable Components和React.lazy的比较" >}}
{{< /style >}}

它的使用方法基本就是把上面的`React.lazy`函数换成`loadable`函数。该函数还可以接收第二个参数，其中可以指定组件加载时的fallback UI，功能类似于`Suspense`（按照文档<https://loadable-components.com/docs/fallback/#fallback-without-suspense>的说法，另一种指定fallback的方式是在JSX中指定，然而实际这样做时会报错）。第一个示例的`Loadable Components`实现方法如下：

```jsx
// Author: Zhiyang Li
// Date: 2020.09.22

import React from 'react';
import loadable from '@loadable/component';

class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      currentComponent: null,
    };
  }

  getComponent() {
    const { currentComponent } = this.state;

    if (currentComponent === null) {
      return null;
    } else if (currentComponent === "OtherComponent") {
      return loadable(
        () => import(/* webpackChunkName: "OtherComponent" */ "./OtherComponent"),
        { fallback: <div>Loading...</div> }
      );
    } else if (currentComponent === "AnotherComponent") {
      return loadable(
        () => import(/* webpackChunkName: "AnotherComponent" */ "./AnotherComponent"),
        { fallback: <div>Loading...</div> }
      );
    }
  }

  render() {
    return (
      <div>
        <div><Component show={() => this.getComponent()} /></div>
        <button onClick={() => this.setState({ currentComponent: "OtherComponent" })}>
          Show OtherComponent
        </button>
        <button onClick={() => this.setState({ currentComponent: "AnotherComponent" })}>
          Show AnotherComponent
        </button>
        <button onClick={() => this.setState({ currentComponent: null })}>
          Clear
        </button>
      </div>
    );
  }
}

function Component(props) {
  let ComponentToShow = props.show();

  if (ComponentToShow === null) {
    return null;
  } else {
    return <ComponentToShow />
  }
}

export default App;
```

## 懒加载与React Router

当我们使用React Router构建单页应用时，可以把除首页以外的页面通过上述方式进行懒加载，这样可以加快首页的响应时间。下面是一个比较简单的在React Router中使用懒加载的例子。整个应用包含三个视图：首页Index（"/"），Page1（"/page1"）和Page2（"/page2"）。首先，组件`App`代表整个应用，它会渲染三个视图的链接和当前视图的内容，如下所示：

App.js

```jsx
// Author: Zhiyang Li
// Date: 2020.09.22

import React from 'react';
import { BrowserRouter, Link, Route, Switch } from 'react-router-dom';
import Index from './Index';
import Page1Wrapper from './Page1Wrapper';
import Page2Wrapper from './Page2Wrapper';

function App() {
    return (
        <BrowserRouter>
            <div>
                <nav>
                    <ul>
                        <li><Link to="/">To Index</Link></li>
                        <li><Link to="/page1">To Page1</Link></li>
                        <li><Link to="/page2">To Page2</Link></li>
                    </ul>
                </nav>

                <hr />

                <Switch>
                    <Route path="/page1" component={Page1Wrapper} />
                    <Route path="/page2" component={Page2Wrapper} />
                    <Route path="/" component={Index} />
                </Switch>
            </div>
        </BrowserRouter>
    );
}

export default App;
```

三个视图对应的组件分别位于模块`Index.js`，`Page1.js`和`Page2.js`中。`Index.js`不需要懒加载，因此在`App.js`中通过静态方式引入。而`Page1.js`和`Page2.js`则通过懒加载的方式引入。为此，我写了两个Wrapper Component，代码如下：

Page1Wrapper.js

```jsx
// Author: Zhiyang Li
// Date: 2020.09.22

import React, { Suspense } from 'react';

const Page1 = React.lazy(() => import(/* webpackChunkName: "page1" */ "./Page1"));

function Page1Wrapper() {
    return (
        <Suspense fallback={<div>Loading Page1...</div>}>
            <Page1 />
        </Suspense>
    );
}

export default Page1Wrapper;
```

Page2Wrapper.js

```jsx
// Author: Zhiyang Li
// Date: 2020.09.22

import React, { Suspense } from 'react';

const Page2 = React.lazy(() => import(/* webpackChunkName: "page2" */ "./Page2"));

function Page2Wrapper() {
    return (
        <Suspense fallback={<div>Loading Page2...</div>}>
            <Page2 />
        </Suspense>
    );
}

export default Page2Wrapper;
```

然后在`App.js`中静态引入`Page1Wrapper.js`和`Page1Wrapper.js`。这样当我们访问首页的时候，`Page1.js`和`Page2.js`对应的模块不会加载，只有访问Page1时才会加载`Page1.js`对应的模块，访问Page2时才会加载`Page2.js`对应的模块，如下图所示：

{{< style "text-align: center;" >}}
{{< image src="/images/react-lazy-load/lazy-with-router.png" caption="React Router中使用懒加载" alt="React Router中使用懒加载" title="React Router中使用懒加载" >}}
{{< /style >}}

## 参考资料

1. <https://webpack.js.org/guides/lazy-loading/>
2. <https://reactjs.org/docs/code-splitting.html#reactlazy>
3. <https://loadable-components.com/docs/getting-started/>
4. <https://reactrouter.com/web/guides/code-splitting>
