# 9 - Redux

在本章（应该是目前为止最困难的一章）中，我们将添加 [Redux](http://redux.js.org/) 到我们的应用程序，并将它与 React 结合起来使用。 Redux 负责管理应用程序的状态。它由 **store**，**actions** 和 **reducers** 组成：**store** 是一个表示应用程序状态的 JavaScript 对象，**actions** 表示由用户触发的动作，**reducers** 表示如何处理这些动作。 reducers 将会改变应用程序的状态（store），当状态被修改时，应用程序可能会发生一些改变。[这里](http://slides.com/jenyaterpil/redux-from-twitter-hype-to-production#/9)有一个很好的 Redux 可视化演示示例。

为了演示如何以最简单的方式使用 Redux，我们的应用程序将包括一个消息和一个按钮。消息的内容是狗是否已经叫了（初始值是没有叫），按钮用于让狗叫，点击按钮后应该更新消息的内容。

需要两个包，`redux` 和 `react-redux`。

- 运行 `yarn add redux react-redux`

新建两个文件夹：`src/client/actions` 和 `src/client/reducers`

- 在 `actions` 中新建 `dog-actions.js`：

```javascript
export const MAKE_BARK = 'MAKE_BARK';

export const makeBark = () => ({
  type: MAKE_BARK,
  payload: true,
});
```

这里我们定义了一个 action 类型 `MAKE_BARK` 和一个函数（也称为 *action creator*），它触发一个名为 `makeBark` 的 `MAKE_BARK` action。两者都使用 `export` 导出了，因为在其他文件中需要它们。此操作实现了 [Flux Standard Action](https://github.com/acdlite/flux-standard-action) 模型，所以它具有 `type` 和 `payload` 属性。

- 在 `reducers` 文件夹中新建 `dog-reducer.js`：

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

这里我们定义了应用程序的初始状态 `initialState` 和 `dogReducer`。`initialState` 是一个 `hasBarked` 属性为 `false` 的 JavaScript 对象，`dogReducer` 是根据 action 改变 state 的函数。在这个函数中不能直接修改 state，而应该返回一个新的 state。

- 修改 `app.jsx` 创建一个 *store*，替换成以下内容：

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

可以看到，store 是由一个叫 `createStore` 的函数生成的。通过使用 `combineReducers` 函数将所有 reducer 结合起来传给 `createStore`。每个 reducer 都可以重新命名，在这里将其命名为 `dog`。

这是 Redux 的部分。

现在我们将使用 `react-redux` 将 Redux 和 React 结合起来。为了将我们的 store 传给 React，需要将整个 app 用 `<Provider>` 组件包裹起来。它只有能有一个子组件，所以我们用一个 `<div>` 将我们的两个组件 `BarkMessage` 和 `BarkButton` 再包了一层。

可以看到，`BarkMessage` 和 `BarkButton` 是从 `containers` 文件夹中导入的。现在我们将介绍 **展示组件 **和 **容器组件** 的概念。

*展示组件* 是 *笨拙的* React 组件，它们无法知道任何的 Redux 状态。*容器组件* 是 *聪明的* 组件，它能获取 Redux 的状态，我们通过使用 `connect` 方法将这些状态传给 *展示组件*。

译者注：关于容器组件（Smart/Container Components）和展示组件（Dumb/Presentational Components）的可能有不同的名称，可以查看 Redux 中文文档的[相关章节](http://cn.redux.js.org/docs/basics/UsageWithReact.html)。

- 新建两个文件夹 `src/client/components` 和 `src/client/containers`
- 在 `components` 中新建以下文件：

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

这些都是**展示组件**的例子，他们并没有跟应用程序的 state 关联起来，而是根据传入的 **props** 来展示内容。`button.jsx` 和 `message.jsx` 有一点区别，`Button` 的 props 里有一个 **action**，绑定在 `onClick` 事件上。在应用程序的上下文中，`Button` 标签不会改变（因为它没有绑定任何 state），但是 `Message` 组件将反映我们的应用程序的状态（绑定了 `message` 这个 state），并将根据状态而变化。

同样，*展示组件*不会知道关于 Redux **actions** 或 **state** 的任何信息，所以我们需要新建一个**容器组件**将 *动作*（actions） 和 *数据*（data）传给它们。

- 在 `containers` 中新建以下文件：

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

在 `BarkButton` 中， 我们将 `Button` 组件和 `makeBark` 这个 action 以及 Redux 中的 `dispatch` 方法关联起来，将 `BarkMessage` 组件和 app 状态中的 `Message` 关联起来。当状态改变的时候，`Message` 组件会自动重新渲染（因为它的 props `message` 改变了）。这些操作都是由 `react-redux` 中的 `connect` 方法完成的。

- 现在可以执行 `yarn start` ，并打开 `index.html` 了。可以看到 "The dog did not bark" 的消息以及一个按钮。点击按钮后消息会变成 "The dog barked"。

译者注：本章概念比较多，可能不太容易理解，如果你希望深入地学习 Redux，请查看 [Redux 中文文档](http://cn.redux.js.org/)

下一章：[10 - Immutable JS 和 Redux 的改进方法](/tutorial/10-immutable-redux-improvements)

回到[上一章](/tutorial/8-react)或[目录](https://github.com/pd4d10/js-stack-from-scratch)
