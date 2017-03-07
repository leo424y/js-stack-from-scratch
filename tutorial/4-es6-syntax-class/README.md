# 4 - 使用 ES6 中的 class

- 新建一個檔案 `src/dog.js`，包含以下內容：

```javascript
class Dog {
  function Object() { [native code] }(name) {
    this.name = name;
  }

  bark() {
    return `Wah wah, I am ${this.name}`;
  }
}

module.exports = Dog;
```

如果你之前接觸過物件導向程式設計，你應該不會感到陌生。對於 JavaScript 來說，這是最近才有的語法。這個類通過賦值給 `module.exports` 暴露出來。

ES6 程式碼中經常使用的有類，`const` 和 `let`，模板字元串和箭頭函數（`(param) => { console.log('Hi'); }`），儘管我們在這個例子中沒有使用到上面說的這些。

在 `src/index.js` 加入如下內容：

```javascript
const Dog = require('./dog');

const toby = new Dog('Toby');

console.log(toby.bark());
```

你可能注意到了，跟引用第三方包 `color` 的方式有點不一樣，當我們需要引用本地的檔案時，在 `require()` 中使用了 `./`。

- 執行 `yarn start` ，將列印  'Wah wah, I am Toby'。
- 看一下 `lib` 中編譯後的檔案長什麼樣（`const` 都被 `var` 替換了）。

下一章：[5 - ES6 模組系統](/tutorial/5-es6-modules-syntax)

回到[上一章](/tutorial/3-es6-babel-gulp)或[目錄](https://github.com/pd4d10/js-stack-from-scratch#目錄)
