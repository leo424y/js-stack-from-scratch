# 10 - Immutable JS 和 Redux 的改進方法

## Immutable JS

與前一章不同，這一章相當容易，只是對之前的程式碼稍作改進。

首先向程式碼庫新增 Immutable JS。Immutable 的中文意思是不可變，Immutable 是一個操作 JS 物件而不改變它們的庫，不是這樣做：

```javascript
const obj = { a: 1 };
obj.a = 2; // Mutates `obj`
```

而是這樣做：

```javascript
const obj = Immutable.Map({ a: 1 });
obj.set('a', 2); // Returns a new object without mutating `obj`
```

這是一種**函數語言程式設計**的範例，它能很好地與 Redux 一起使用。reducer 函數*必須*是純函數，不能改變作為參數傳遞的 state，而是返回一個全新的 state 物件。讓我們使用 Immutable 來強制做到這一點。

- 執行 `yarn add immutable`

我們將使用 `Map`，但 Airbnb 配置的 ESLint 會將其視為一個錯誤（大寫開頭的變數必須是一個類）。新增以下程式碼到 `package.json` 的 `eslintConfig`：

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

這樣 `Map` 和 `List`（我們使用到的兩個 Immutable 物件）就不會被 ESLint 視為錯誤了。這個 JSON 的實際上是被 Yarn/NPM 自動格式化了，所以我們沒辦法讓它更緊湊。

回到 Immutable 的部分：

修改 `dog-reducer.js` 檔案如下：

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

初始 state 現在是一個 Immutable Map 了，使用 `set()` 方法生成一個新的 state，這就避免了改變原來的 state 物件。

在 `containers/bark-message.js` 中的 `mapStateToProps` 函數將 `.hasBarked` 替換為 `.get('hasBarked')`：

```javascript
const mapStateToProps = state => ({
  message: state.dog.get('hasBarked') ? 'The dog barked' : 'The dog did not bark',
});
```

修改後 app 的行為與之前是一樣的。

**注意**：如果 Babel 提示 Immutable 超過了 100KB，在 `package.json` 中的 `babel` 欄位新增 `"compact": false`。

從上面的程式碼片段中可以看出，我們的 state 物件 `dog` 屬性仍然不是不可變的。這沒問題，但如果你只想操作不可變物件，可以安裝 `redux-immutable` 包來替換 Redux 的 `combineReducers` 函數。

**可選步驟**：

- 執行 `yarn add redux-immutable`
- 把 `app.jsx` 中的 `combineReducers` 函數替換為從 `redux-immutable` 中匯出的。
- 將 `bark-message.js` 中的 `state.dog.get('hasBarked')` 替換為 `state.getIn(['dog', 'hasBarked'])`。

## Redux Actions

當你新增越來越多的 action 後，你會發現自己寫了很多重複程式碼。 `redux-actions` 這個包有助於減少重複的樣板程式碼，你可以以更緊湊的方式重寫 `dog-actions.js` 檔案：

```javascript
import { createAction } from 'redux-actions';

export const MAKE_BARK = 'MAKE_BARK';
export const makeBark = createAction(MAKE_BARK, () => true);
```

`redux-actions` 實現了 [Flux Standard Action](https://github.com/acdlite/flux-standard-action) 這個模型，就跟我們之前的程式碼一樣，所以它能夠無縫地整合在我們的 app 中。

- 不要忘記執行 `yarn add redux-actions`

下一章：[11 - 使用 Mocha, Chai 和 Sinon 進行測試](/tutorial/11-testing-mocha-chai-sinon)

回到[上一章](/tutorial/9-redux)或[目錄](https://github.com/pd4d10/js-stack-from-scratch#目錄)
