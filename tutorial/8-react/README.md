# 8 - React

我們現在要使用 React 來渲染我們的應用程式。

首先安裝 React 和 ReactDOM：

- 執行 `yarn add react react-dom`

這兩個包將新增到 `"dependencies"` 而不是 `"devDependencies"` ，因為生產環境的客戶端打包需要它們，這跟構建過程需要的包不一樣。

將 `src/client/app.js` 重新命名為 `src/client/app.jsx`，然後加入如下程式碼：

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

**注意**：如果您不熟悉 React 或 PropTypes 相關的知識，請先學習 React 的基本知識後再回到本教程。在接下來的章節裡會有很多關於 React 的內容，所以你需要先理解它。

在你的 Gulpfile 中修改 `clientEntryPoint` 的值，改成 `.jsx` 副檔名：

```javascript
clientEntryPoint: 'src/client/app.jsx',
```

我們使用了 JSX 語法，必須告訴 Babel 需要轉換它。首先安裝 React Babel preset，這個包告訴 Babel 如何轉換 JSX 語法：`yarn add --dev babel-preset-react`。然後修改 `package.json` 中的 `babel` 欄位：

```json
"babel": {
  "presets": [
    "latest",
    "react"
  ]
},
```

現在執行 `yarn start`，開啟 `index.html`，可以看見 React 渲染出了 "The dog says: Wah wah, I am Browser Toby"。

下一章：[9 - Redux](/tutorial/9-redux)

返回[上一章](/tutorial/7-client-webpack)或[目錄](https://github.com/pd4d10/js-stack-from-scratch#目錄)
