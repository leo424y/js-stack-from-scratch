# 9 - Redux

在本章（應該是目前為止最困難的一章）中，我們將新增 [Redux](http://redux.js.org/) 到我們的應用程式，並將它與 React 結合起來使用。 Redux 負責管理應用程式的狀態。它由 **store**，**actions** 和 **reducers** 組成：**store** 是一個表示應用程式狀態的 JavaScript 物件，**actions** 表示由使用者觸發的動作，**reducers** 表示如何處理這些動作。 reducers 將會改變應用程式的狀態（store），當狀態被修改時，應用程式可能會發生一些改變。[這裡](http://slides.com/jenyaterpil/redux-from-twitter-hype-to-production#/9)有一個很好的 Redux 視覺化演示示例。

為了演示如何以最簡單的方式使用 Redux，我們的應用程式將包括一個訊息和一個按鈕。訊息的內容是狗是否已經叫了（初始值是沒有叫），按鈕用於讓狗叫，點選按鈕後應該更新訊息的內容。

需要兩個包，`redux` 和 `react-redux`。

- 執行 `yarn add redux react-redux`

新建兩個資料夾：`src/client/actions` 和 `src/client/reducers`

- 在 `actions` 中新建 `dog-actions.js`：

```javascript
export const MAKE_BARK = 'MAKE_BARK';

export const makeBark = () => ({
  type: MAKE_BARK,
  payload: true,
});
```

這裡我們定義了一個 action 類型 `MAKE_BARK` 和一個函數（也稱為 *action creator*），它觸發一個名為 `makeBark` 的 `MAKE_BARK` action。兩者都使用 `export` 匯出了，因為在其他檔案中需要它們。此操作實現了 [Flux Standard Action](https://github.com/acdlite/flux-standard-action) 模型，所以它具有 `type` 和 `payload` 屬性。

- 在 `reducers` 資料夾中新建 `dog-reducer.js`：

```javascript
import { MAKE_BARK } from '../actions/dog-actions';

const initialState = {
  hasBarked: false,
};

const dogReducer = (state = initialState, action) => {
  switch (action.type) {
    case MAKE_BARK:
      return { hasBarked: action.payload };
    default:
      return state;
  }
};

export default dogReducer;
```

這裡我們定義了應用程式的初始狀態 `initialState` 和 `dogReducer`。`initialState` 是一個 `hasBarked` 屬性為 `false` 的 JavaScript 物件，`dogReducer` 是根據 action 改變 state 的函數。在這個函數中不能直接修改 state，而應該返回一個新的 state。

- 修改 `app.jsx` 建立一個 *store*，替換成以下內容：

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import { createStore, combineReducers } from 'redux';
import { Provider } from 'react-redux';
import dogReducer from './reducers/dog-reducer';
import BarkMessage from './containers/bark-message';
import BarkButton from './containers/bark-button';

const store = createStore(combineReducers({
  dog: dogReducer,
}));

ReactDOM.render(
  <Provider store={store}>
    <div>
      <BarkMessage />
      <BarkButton />
    </div>
  </Provider>
  , document.querySelector('.app')
);
```

可以看到，store 是由一個叫 `createStore` 的函數生成的。通過使用 `combineReducers` 函數將所有 reducer 結合起來傳給 `createStore`。每個 reducer 都可以重新命名，在這裡將其命名為 `dog`。

這是 Redux 的部分。

現在我們將使用 `react-redux` 將 Redux 和 React 結合起來。為了將我們的 store 傳給 React，需要將整個 app 用 `<Provider>` 元件包裹起來。它只有能有一個子元件，所以我們用一個 `<div>` 將我們的兩個元件 `BarkMessage` 和 `BarkButton` 再包了一層。

可以看到，`BarkMessage` 和 `BarkButton` 是從 `containers` 資料夾中匯入的。現在我們將介紹 **展示元件 **和 **容器元件** 的概念。

*展示元件* 是 *笨拙的* React 元件，它們無法知道任何的 Redux 狀態。*容器元件* 是 *聰明的* 元件，它能獲取 Redux 的狀態，我們通過使用 `connect` 方法將這些狀態傳給 *展示元件*。

譯者注：關於容器元件（Smart/Container Components）和展示元件（Dumb/Presentational Components）的可能有不同的名稱，可以檢視 Redux 中文文件的[相關章節](http://cn.redux.js.org/docs/basics/UsageWithReact.html)。

- 新建兩個資料夾 `src/client/components` 和 `src/client/containers`
- 在 `components` 中新建以下檔案：

**button.jsx**

```javascript
import React, { PropTypes } from 'react';

const Button = ({ action, actionLabel }) => <button onClick={action}>{actionLabel}</button>;

Button.propTypes = {
  action: PropTypes.func.isRequired,
  actionLabel: PropTypes.string.isRequired,
};

export default Button;
```

以及 **message.jsx**:

```javascript
import React, { PropTypes } from 'react';

const Message = ({ message }) => <div>{message}</div>;

Message.propTypes = {
  message: PropTypes.string.isRequired,
};

export default Message;
```

這些都是**展示元件**的例子，他們並沒有跟應用程式的 state 關聯起來，而是根據傳入的 **props** 來展示內容。`button.jsx` 和 `message.jsx` 有一點區別，`Button` 的 props 裡有一個 **action**，繫結在 `onClick` 事件上。在應用程式的上下文中，`Button` 標籤不會改變（因為它沒有繫結任何 state），但是 `Message` 元件將反映我們的應用程式的狀態（繫結了 `message` 這個 state），並將根據狀態而變化。

同樣，*展示元件*不會知道關於 Redux **actions** 或 **state** 的任何資訊，所以我們需要新建一個**容器元件**將 *動作*（actions） 和 *資料*（data）傳給它們。

- 在 `containers` 中新建以下檔案：

**bark-button.js**

```javascript
import { connect } from 'react-redux';
import Button from '../components/button';
import { makeBark } from '../actions/dog-actions';

const mapDispatchToProps = dispatch => ({
  action: () => { dispatch(makeBark()); },
  actionLabel: 'Bark',
});

export default connect(null, mapDispatchToProps)(Button);
```

以及 **bark-message.js**:

```javascript
import { connect } from 'react-redux';
import Message from '../components/message';

const mapStateToProps = state => ({
  message: state.dog.hasBarked ? 'The dog barked' : 'The dog did not bark',
});

export default connect(mapStateToProps)(Message);
```

在 `BarkButton` 中， 我們將 `Button` 元件和 `makeBark` 這個 action 以及 Redux 中的 `dispatch` 方法關聯起來，將 `BarkMessage` 元件和 app 狀態中的 `Message` 關聯起來。當狀態改變的時候，`Message` 元件會自動重新渲染（因為它的 props `message` 改變了）。這些操作都是由 `react-redux` 中的 `connect` 方法完成的。

- 現在可以執行 `yarn start` ，並開啟 `index.html` 了。可以看到 "The dog did not bark" 的訊息以及一個按鈕。點選按鈕後訊息會變成 "The dog barked"。

譯者注：本章概念比較多，可能不太容易理解，如果你希望深入地學習 Redux，請檢視 [Redux 中文文件](http://cn.redux.js.org/)

下一章：[10 - Immutable JS 和 Redux 的改進方法](/tutorial/10-immutable-redux-improvements)

回到[上一章](/tutorial/8-react)或[目錄](https://github.com/pd4d10/js-stack-from-scratch#目錄)
