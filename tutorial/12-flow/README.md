# 12 - Flow

[Flow](https://flowtype.org/) 是一個靜態類型檢查器。它檢查程式碼中的類型問題，可以通過註解（annotation）新增顯式類型聲明。

- 為了讓 Babel 轉換的時候移除 Flow 的註解，請執行 `yarn add --dev babel-preset-flow` 安裝 Babel 的 Flow preset。然後在 `package.json` 中的 `babel.presets` 下新增 `"flow"`。
- 在根目錄新建 `.flowconfig` 檔案
- 執行 `yarn add --dev gulp-flowtype` 安裝 Flow 的 Gulp 外掛，在 `lint` 任務中新增 `flow()`，如下：

```javascript
import flow from 'gulp-flowtype';

// [...]

gulp.task('lint', () =>
  gulp.src([
    paths.allSrcJs,
    paths.gulpFile,
    paths.webpackFile,
  ])
    .pipe(eslint())
    .pipe(eslint.format())
    .pipe(eslint.failAfterError())
    .pipe(flow({ abort: true })) // Add Flow here
);
```

`abort` 選項用於在檢查到錯誤時中斷 Gulp 任務。

現在應該可以執行 Flow 了。

- 在 `src/shared/dog.js` 中新增 Flow 註解如下：

```javascript
// @flow

class Dog {
  name: string;

  function Object() { [native code] }(name: string) {
    this.name = name;
  }

  bark(): string {
    return `Wah wah, I am ${this.name}`;
  }

  barkInConsole() {
    /* eslint-disable no-console */
    console.log(this.bark());
    /* eslint-enable no-console */
  }

}

export default Dog;
```

`// @flow` 註釋告訴 Flow 這個檔案需要進行類型檢查。Flow 註解通常是在函數參數或函數名後加一個冒號。如果需要進一步瞭解，可以檢視文件。

現在執行 `yarn start`，Flow 是正常的，但 ESLint 將會提示程式碼的語法有錯。因為我們裝了 `babel-preset-flow`，所以 Babel 能正常地解析 Flow 語法，而 ESLint 不能，如果 ESLint 能通過 Babel 的解析器來工作就好了。沒問題，只要安裝 `babel-eslint` 這個包就好了：

- 執行 `yarn add --dev babel-eslint`

- 在 `package.json` 的 `eslintConfig` 欄位新增 `"parser": "babel-eslint"`

`yarn start`，現在程式碼檢查和類型檢查應該都正常了。

現在 ESLint 和 Babel 共享同一個程式碼解析器，可以通過 `eslint-plugin-flowtype` 外掛來檢查 Flow 註解了。

- 執行 `yarn add --dev eslint-plugin-flowtype`，並在 `package.json` 中的 `eslintConfig.plugins` 新增 `"flowtype"`，向 `eslintConfig.extends` 陣列新增 `"plugin:flowtype/recommended"`

現在，如果你使用 `name:string` 作為註釋，ESLint 會提示冒號後少了一個空格。

**注意**：在 `package.json` 中我們新增的 `"parser": "babel-eslint"` 屬性包含了 `"plugin:flowtype/recommended"` 配置，所以實際上你可以移除它，讓 `package.json` 更簡潔。保留它則讓語義顯得更明確，這取決於你的個人喜好。本教程旨在最簡潔的配置檔案，所以我將它刪除了。

- 現在可以在 `src` 目錄下的每個 `.js` 和 `.jsx` 檔案中新增 `// @flow` 註釋了，執行 `yarn test` 或 `yarn start`， 在每個 Flow 提示你的地方新增註解。

在 `src/client/components/message.jsx` 檔案中，有一種比較違反直覺的情況如下：

```javascript
const Message = ({ message }: { message: string }) => <div>{message}</div>;
```

在解構函數參數時，必須使用物件字面量來新增註解。

另一個可能會遇到的情況是，在 `src/client/reducers/dog-reducer.js` 中，Flow 會提示 Immutable 沒有 export default。這個問題在 [#863 on Immutable](https://github.com/facebook/immutable-js/issues/863) 中提出了兩個解決方法：

```javascript
import { Map as ImmutableMap } from 'immutable';
// or
import * as Immutable from 'immutable';
```

在 Immutable 正式解決這個問題之前，選擇你喜歡的方式吧。我個人傾向於使用 `import * as Immutable from 'immutable'`，因為它更短，而且這個問題修復以後不需要更改也能正常執行。

**注意**：如果 Flow 在 `node_modules` 資料夾中檢測到類型錯誤，請在 `.flowconfig` 中新增一個`[ignore]` 來忽略特定的包（不要忽略整個 `node_modules` 目錄）。看起來像這樣

```flowconfig
[ignore]

.*/node_modules/gulp-flowtype/.*
```

在我這的情況是，Atom 編輯器的 `linter-flow` 外掛會檢測 `node_modules/gulp-flowtype` 目錄中的類型錯誤，針對那些用 `// @flow` 註釋過的檔案。

現在，你的程式碼已經通過了程式碼檢查，類型檢查和測試的考驗了，好樣的！

回到[上一章](/tutorial/11-testing-mocha-chai-sinon)或[目錄](https://github.com/pd4d10/js-stack-from-scratch#目錄)
