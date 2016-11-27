# 10 - Immutable JS 和 Redux 的改进方法

## Immutable JS

与前一章不同，这一章相当容易，只是对之前的代码稍作改进。

首先向代码库添加 Immutable JS。Immutable 的中文意思是不可变，Immutable 是一个操作 JS 对象而不改变它们的库，不是这样做：

```javascript
const obj = { a: 1 };
obj.a = 2; // Mutates `obj`
```

而是这样做：

```javascript
const obj = Immutable.Map({ a: 1 });
obj.set('a', 2); // Returns a new object without mutating `obj`
```

这是一种**函数式编程**的范例，它能很好地与 Redux 一起使用。reducer 函数*必须*是纯函数，不能改变作为参数传递的 state，而是返回一个全新的 state 对象。让我们使用 Immutable 来强制做到这一点。

- 运行 `yarn add immutable`

我们将使用 `Map`，但 Airbnb 配置的 ESLint 会将其视为一个错误（大写开头的变量必须是一个类）。添加以下代码到 `package.json` 的 `eslintConfig`：

```json
"rules": {
  "new-cap": [
    2,
    {
      "capIsNewExceptions": [
        "Map",
        "List"
      ]
    }
  ]
}
```

这样 `Map` 和 `List`（我们使用到的两个 Immutable 对象）就不会被 ESLint 视为错误了。这个 JSON 的实际上是被 Yarn/NPM 自动格式化了，所以我们没办法让它更紧凑。

回到 Immutable 的部分：

修改 `dog-reducer.js` 文件如下：

```javascript
import Immutable from 'immutable';
import { MAKE_BARK } from '../actions/dog-actions';

const initialState = Immutable.Map({
  hasBarked: false,
});

const dogReducer = (state = initialState, action) => {
  switch (action.type) {
    case MAKE_BARK:
      return state.set('hasBarked', action.payload);
    default:
      return state;
  }
};

export default dogReducer;
```

初始 state 现在是一个 Immutable Map 了，使用 `set()` 方法生成一个新的 state，这就避免了改变原来的 state 对象。

在 `containers/bark-message.js` 中的 `mapStateToProps` 函数将 `.hasBarked` 替换为 `.get('hasBarked')`：

```javascript
const mapStateToProps = state => ({
  message: state.dog.get('hasBarked') ? 'The dog barked' : 'The dog did not bark',
});
```

修改后 app 的行为与之前是一样的。

**注意**：如果 Babel 提示 Immutable 超过了 100KB，在 `package.json` 中的 `babel` 字段添加 `"compact": false`。

从上面的代码片段中可以看出，我们的 state 对象 `dog` 属性仍然不是不可变的。这没问题，但如果你只想操作不可变对象，可以安装 `redux-immutable` 包来替换 Redux 的 `combineReducers` 函数。

**可选步骤**：

- 运行 `yarn add redux-immutable`
- 把 `app.jsx` 中的 `combineReducers` 函数替换为从 `redux-immutable` 中导出的。
- 将 `bark-message.js` 中的 `state.dog.get('hasBarked')` 替换为 `state.getIn(['dog', 'hasBarked'])`。

## Redux Actions

当你添加越来越多的 action 后，你会发现自己写了很多重复代码。 `redux-actions` 这个包有助于减少重复的样板代码，你可以以更紧凑的方式重写 `dog-actions.js` 文件：

```javascript
import { createAction } from 'redux-actions';

export const MAKE_BARK = 'MAKE_BARK';
export const makeBark = createAction(MAKE_BARK, () => true);
```

`redux-actions` 实现了 [Flux Standard Action](https://github.com/acdlite/flux-standard-action) 这个模型，就跟我们之前的代码一样，所以它能够无缝地集成在我们的 app 中。

- 不要忘记运行 `yarn add redux-actions`

下一章：[11 - 使用 Mocha, Chai 和 Sinon 进行测试](/tutorial/11-testing-mocha-chai-sinon)

回到[上一章](/tutorial/9-redux)或[目录](https://github.com/pd4d10/js-stack-from-scratch#目录)
