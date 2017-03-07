# 11 - 使用 Mocha, Chai 和 Sinon 進行測試

## Mocha 和 Chai

- 新建 `src/test` 資料夾。這個資料夾的目錄結構應該與 app 的目錄結構一致，所以也應該新建 `src/test/client` 資料夾（可以不新增 `server` and `shared`，目前我們只寫這兩個資料夾下的測試）
- 在 `src/test/client` 中新建 `state-test.js`，我們將在這裡編寫 Redux 生命週期的測試用例。

我們選擇 [Mocha](http://mochajs.org/) 作為主要的測試框架。Mocha 易於使用，功能強大，是目前[最流行的 JavaScript 測試框架](http://stateofjs.com/2016/testing/)，靈活並且模組化。另外，它允許你使用任何的斷言庫。 Chai 是一個非常棒的斷言庫，有很多[外掛](http://chaijs.com/plugins/)可用，並允許你選擇不同的斷言樣式。`

- 安裝 Mocha 和 Chai：執行 `yarn add --dev mocha chai`

在 `state-test.js` 中新增：

```javascript
/* eslint-disable import/no-extraneous-dependencies, no-unused-expressions */

import { createStore } from 'redux';
import { combineReducers } from 'redux-immutable';
import { should } from 'chai';
import { describe, it, beforeEach } from 'mocha';
import dogReducer from '../../client/reducers/dog-reducer';
import { makeBark } from '../../client/actions/dog-actions';

should();
let store;

describe('App State', () => {
  describe('Dog', () => {
    beforeEach(() => {
      store = createStore(combineReducers({
        dog: dogReducer,
      }));
    });
    describe('makeBark', () => {
      it('should make hasBarked go from false to true', () => {
        store.getState().getIn(['dog', 'hasBarked']).should.be.false;
        store.dispatch(makeBark());
        store.getState().getIn(['dog', 'hasBarked']).should.be.true;
      });
    });
  });
});
```

好，現在我們一起分析一下都發生了什麼。

首先，注意我們是如何從 `chai` 中匯入 `should` 斷言樣式的。這讓我們使用 `mynumber.should.equal(3)` 這樣的語法去做斷言，很酷。為了能夠讓 `should` 讓任何物件呼叫，需要所有測試之前執行 `should()`。這些斷言中，有些是表示式，如 `mybook.should.be.true`，這會讓 ESLint 報錯，因此我們在頂部新增了一個 ESLint 註釋，用於禁用 `no-unused-expressions` 這個規則。

Mocha 測試的工作就像一棵樹。在這個例子中，`makeBark` 方法會修改 state 中的 `dog` 屬性，我們想測試這個方法，所以應該使用這種層次結構：`App State > Dog > makeBark`，正如上面程式碼裡 `describe()` 聲明的一樣。`it()` 是實際的測試函數， `beforeEach()` 是在每個 `it()` 測試之前執行的函數。在這個例子中，在測試之前我們需要新建一個 store。在檔案的頂部聲明一個 `store` 變數，這樣在每個測試用例中都能用了。

`makeBark` 這個測試的意義非常明確， `it()` 中提供的字元串描述使它更加明確了：這個測試用例是在測試呼叫了 `hasBarked` 方法後 `hasBarked` 將從 `false` 變為 `true`。

好，該執行測試了！

- 在 `gulpfile.babel.js` 中新建 `test` 任務，它依賴 `gulp-mocha` 外掛：

```javascript
import mocha from 'gulp-mocha';

const paths = {
  // [...]
  allLibTests: 'lib/test/**/*.js',
};

// [...]

gulp.task('test', ['build'], () =>
  gulp.src(paths.allLibTests)
    .pipe(mocha())
);
```

- 當然在此之前需要執行 `yarn add --dev gulp-mocha` 安裝它

如你所見，測試需要依賴 `lib` 中的程式碼，所以 `build` 任務是 `test` 任務的依賴。`build` 也有一個依賴，`lint`，最後我們為 `main` 新增依賴 `test`，所以 `default` 任務的依賴順序是這樣的：`lint` > `build` > `test` > `main`。

- 為 `main` 新增依賴 `test`：

```javascript
gulp.task('main', ['test'], () => /* ... */ );
```

- 在 `package.json` 中將 `"test"` 欄位替換為 `"test": "gulp test"`。 這樣就能使用 `yarn test` 命令執行測試了。像持續整合服務等都將預設讀取 `test` 中的內容，這是一種標準的做法，因此我們應該始終在 `test` 中填寫測試命令。`yarn start` 命令將在 Webpack 進行客戶端打包之前執行所有測試，確保了只有所有測試通過才進行打包。
- 執行 `yarn test` 或 `yarn start`，將會展現測試的結果，希望全是綠色（代表測試通過了）。

## Sinon

在某些情況下，我們希望在單元測試中偽造一些東西。假設我們有一個函數 `deleteEverything`，它包含對 `deleteDatabases()` 的呼叫。如果真的執行了 `deleteDatabases()`（刪除資料庫），將會有很多副作用，我們絕不希望在執行測試時發生這種情況。

[Sinon](http://sinonjs.org/) 是一個提供 **Stubs**（還有許多其他功能）的測試庫，它允許我們不實際呼叫 `deleteDatabases` 而是監聽它。這樣我們可以測試它是否被呼叫，或者它呼叫了哪些參數。它可以偽造或避免 AJAX 請求，從而避免了真實 AJAX 請求對後端的副作用。

譯者注：stub 通常翻譯為測試樁。拿 `deleteDatabases` 舉例，我們肯定不希望執行測試把資料庫都幹掉，打了測試樁之後，跑測試用例不會真的刪資料庫，但我們就能知道這個方法是否執行，執行過幾次，執行時傳入的參數是什麼等等。

我們將  `src/shared/dog.js` 的`Dog` 類中增加一個 `barkInConsole` 方法：

```javascript
class Dog {
  function Object() { [native code] }(name) {
    this.name = name;
  }

  bark() {
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

如果在單元測試中執行 `barkInConsole`，`console.log()` 會把資訊列印到終端。我們把它作為一種副作用來看待（就像 AJAX 一樣），不希望它真的把東西列印出來，而是想知道 `console.log()` *是否正常被呼叫*，以及*呼叫的參數是什麼*。

- 新建 `src/test/shared/dog-test.js` 檔案，加入以下內容：

```javascript
/* eslint-disable import/no-extraneous-dependencies, no-console */

import chai from 'chai';
import { stub } from 'sinon';
import sinonChai from 'sinon-chai';
import { describe, it } from 'mocha';
import Dog from '../../shared/dog';

chai.should();
chai.use(sinonChai);

describe('Shared', () => {
  describe('Dog', () => {
    describe('barkInConsole', () => {
      it('should print a bark string with its name', () => {
        stub(console, 'log');
        new Dog('Test Toby').barkInConsole();
        console.log.should.have.been.calledWith('Wah wah, I am Test Toby');
        console.log.restore();
      });
    });
  });
});
```

這裡我們使用了 Sinon 的 *stubs* 功能和一個 Chai 的外掛。

- 執行 `yarn add --dev sinon sinon-chai` 安裝需要的庫檔案

那麼這裡到底做了什麼呢？首先，我們呼叫 `chai.use(sinonChai)` 來啟用 Chai 外掛。所有的奧祕都在發生在 `it()` 語句：`stub(console, 'log')`  將會為 `console.log` 打一個 stab 並監聽它。當 `new Dog('Test Toby').barkInConsole()` 執行的時候，不出意外 `console.log` 應當會被呼叫。我們使用 `console.log.should.have.been.calledWith()` 來測試是否呼叫了 `console.log`。測試結束後恢復 `console.log`，使它再次正常工作。

**重要說明**：不推薦對 `console.log` 使用 stab，因為一旦測試失敗，`console.log.restore()` 就不會被呼叫了，所以在其他測試程式碼執行的時候 `console.log` 一直是壞的！甚至不會列印導致測試失敗的錯誤訊息，這會很麻煩，一旦出了問題我們也不知道是什麼問題。這個例子只用於說明 stub 功能。

如果一切順利，目前應該有兩個 “pass” 的測試用例了。

下一章：[12 - 使用 Flow 進行類型檢查](/tutorial/12-flow)

回到[上一章](/tutorial/10-immutable-redux-improvements)或[目錄](https://github.com/pd4d10/js-stack-from-scratch#目錄)
