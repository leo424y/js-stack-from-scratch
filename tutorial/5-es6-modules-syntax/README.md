# 5 - ES6 模組系統

將 `const Dog = require('./dog')` 替換為 `import Dog from './dog'`，這是 ES6 模組系統的語法（前一種屬於 CommonJS 模組語法）。

將 `dog.js` 中的  `module.exports = Dog` 也替換為 `export default Dog`。

在 `dog.js` 中，`Dog` 這個變數只在 `export` 中使用了。因此我們可以直接匯出一個匿名的類，如下：

```javascript
export default class {
  function Object() { [native code] }(name) {
    this.name = name;
  }

  bark() {
    return `Wah wah, I am ${this.name}`;
  }
}
```

你可能已經猜到了，使用 `import`  匯入 `index.js` 不一定要使用 Dog 來指代它，其他名字也可以，它只是一個代號而已。下面的程式碼也能正常執行：

```javascript
import Cat from './dog';

const toby = new Cat('Toby');
```

大多數時候，我們會使用與 類/模組 相同的名字。在 Gulp 檔案中我們使用 `const babel = require('gulp-babel')`，這是一個反例。

那 `gulpfile.js` 中的那些 `require()` 可以替換成 `import` 嗎？最新版本的 Node 支援大部分的 ES6 功能，但不支援 ES6 模組語法。不過幸運的是，可以讓 Babel 來幫助 Gulp 做這件事。如果我們將 `gulpfile.js` 重新命名為 `gulpfile.babel.js`，Babel 會將 `import` 匯入的模組傳給 Gulp。

- 將 `gulpfile.js` 重新命名為 `gulpfile.babel.js`
- 替換其中的 `require()`：

```javascript
import gulp from 'gulp';
import babel from 'gulp-babel';
import del from 'del';
import { exec } from 'child_process';
```

注意 `exec` 方法直接從 `child_process` 模組中提取出來了（使用了 ES6 的解構賦值語法）。非常優雅！

- `yarn start` 還是會列印 "Wah wah, I am Toby"

下一章：[6 - 程式碼檢查工具 ESLint](/tutorial/6-eslint)

回到[上一章](/tutorial/4-es6-syntax-class)或[目錄](https://github.com/pd4d10/js-stack-from-scratch#目錄)
