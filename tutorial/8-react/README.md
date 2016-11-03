# 8 - React

我们现在要使用 React 来渲染我们的应用程序。

首先安装 React 和 ReactDOM：

- 运行 `yarn add react react-dom`

这两个包将添加到 `"dependencies"` 而不是 `"devDependencies"` ，因为生产环境的客户端打包需要它们，这跟构建过程需要的包不一样。

将 `src/client/app.js` 重命名为 `src/client/app.jsx`，然后加入如下代码：

```javascript
import 'babel-polyfill';

import React, { PropTypes } from 'react';
import ReactDOM from 'react-dom';
import Dog from '../shared/dog';

const dogBark = new Dog('Browser Toby').bark();

const App = props => (
  <div>
    The dog says: {props.message}
  </div>
);

App.propTypes = {
  message: PropTypes.string.isRequired,
};

ReactDOM.render(<App message={dogBark} />, document.querySelector('.app'));
```

**注意**：如果您不熟悉 React 或 PropTypes 相关的知识，请先学习 React 的基本知识后再回到本教程。在接下来的章节里会有很多关于 React 的内容，所以你需要先理解它。

在你的 Gulpfile 中修改 `clientEntryPoint` 的值，改成 `.jsx` 扩展名：

```javascript
clientEntryPoint: 'src/client/app.jsx',
```

我们使用了 JSX 语法，必须告诉 Babel 需要转换它。首先安装 React Babel preset，这个包告诉 Babel 如何转换 JSX 语法：`yarn add --dev babel-preset-react`。然后修改 `package.json` 中的 `babel` 字段：

```json
"babel": {
  "presets": [
    "latest",
    "react"
  ]
},
```

现在运行 `yarn start`，打开 `index.html`，可以看见 React 渲染出了 "The dog says: Wah wah, I am Browser Toby"。

下一章：[9 - Redux](/tutorial/9-redux)

返回[上一章](/tutorial/7-client-webpack)或[目录](https://github.com/pd4d10/js-stack-from-scratch).
