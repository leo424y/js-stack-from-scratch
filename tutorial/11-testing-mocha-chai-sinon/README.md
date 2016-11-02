# 11 - 使用 Mocha, Chai 和 Sinon 进行测试

## Mocha 和 Chai

- 新建 `src/test` 文件夹。这个文件夹的目录结构应该与 app 的目录结构一致，所以也应该新建 `src/test/client` 文件夹（可以不添加 `server` and `shared`，目前我们只写这两个文件夹下的测试）
- 在 `src/test/client` 中新建 `state-test.js`，我们将在这里编写 Redux 生命周期的测试用例。

我们选择 [Mocha](http://mochajs.org/) 作为主要的测试框架。Mocha 易于使用，功能强大，是目前[最流行的 JavaScript 测试框架](http://stateofjs.com/2016/testing/)，灵活并且模块化。另外，它允许你使用任何的断言库。 Chai 是一个非常棒的断言库，有很多[插件](http://chaijs.com/plugins/)可用，并允许你选择不同的断言样式。`

- 安装 Mocha 和 Chai：运行 `yarn add --dev mocha chai`

在 `state-test.js` 中添加：

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
好，现在我们一起分析一下都发生了什么。

首先，注意我们是如何从 `chai` 中导入 `should` 断言样式的。这让我们使用 `mynumber.should.equal(3)` 这样的语法去做断言，很酷。为了能够让 `should` 让任何对象调用，需要所有测试之前运行 `should()`。这些断言中，有些是表达式，如 `mybook.should.be.true`，这会让 ESLint 报错，因此我们在顶部添加了一个 ESLint 注释，用于禁用 `no-unused-expressions` 这个规则。

Mocha 测试的工作就像一棵树。在这个例子中，`makeBark` 方法会修改 state 中的 `dog` 属性，我们想测试这个方法，所以应该使用这种层次结构：`App State > Dog > makeBark`，正如上面代码里 `describe()` 声明的一样。`it()` 是实际的测试函数， `beforeEach()` 是在每个 `it()` 测试之前执行的函数。在这个例子中，在测试之前我们需要新建一个 store。在文件的顶部声明一个 `store` 变量，这样在每个测试用例中都能用了。

`makeBark` 这个测试的意义非常明确， `it()` 中提供的字符串描述使它更加明确了：这个测试用例是在测试调用了 `hasBarked` 方法后 `hasBarked` 将从 `false` 变为 `true`。

好，该运行测试了！

- 在 `gulpfile.babel.js` 中新建 `test` 任务，它依赖 `gulp-mocha` 插件：

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

- 当然在此之前需要运行 `yarn add --dev gulp-mocha` 安装它

如你所见，测试需要依赖 `lib` 中的代码，所以 `build` 任务是 `test` 任务的依赖。`build` 也有一个依赖，`lint`，最后我们为 `main` 添加依赖 `test`，所以 `default` 任务的依赖顺序是这样的：`lint` > `build` > `test` > `main`。

- 为 `main` 添加依赖 `test`：

```javascript
gulp.task('main', ['test'], () => /* ... */ );
```

- 在 `package.json` 中将 `"test"` 字段替换为 `"test": "gulp test"`。 这样就能使用 `yarn test` 命令运行测试了。像持续集成服务等都将默认读取 `test` 中的内容，这是一种标准的做法，因此我们应该始终在 `test` 中填写测试命令。`yarn start` 命令将在 Webpack 进行客户端打包之前运行所有测试，确保了只有所有测试通过才进行打包。
- 运行 `yarn test` 或 `yarn start`，将会展现测试的结果，希望全是绿色（代表测试通过了）。

## Sinon

在某些情况下，我们希望在单元测试中伪造一些东西。假设我们有一个函数 `deleteEverything`，它包含对 `deleteDatabases()` 的调用。如果真的运行了 `deleteDatabases()`（删除数据库），将会有很多副作用，我们绝不希望在运行测试时发生这种情况。

[Sinon](http://sinonjs.org/) 是一个提供 **Stubs**（还有许多其他功能）的测试库，它允许我们不实际调用 `deleteDatabases` 而是监听它。这样我们可以测试它是否被调用，或者它调用了哪些参数。它可以伪造或避免 AJAX 请求，从而避免了真实 AJAX 请求对后端的副作用。

译者注：stub 通常翻译为测试桩。拿 `deleteDatabases` 举例，我们肯定不希望运行测试把数据库都干掉，打了测试桩之后，跑测试用例不会真的删数据库，但我们就能知道这个方法是否执行，执行过几次，执行时传入的参数是什么等等。

我们将  `src/shared/dog.js` 的`Dog` 类中增加一个 `barkInConsole` 方法：

```javascript
class Dog {
  constructor(name) {
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

如果在单元测试中运行 `barkInConsole`，`console.log()` 会把信息打印到终端。我们把它作为一种副作用来看待（就像 AJAX 一样），不希望它真的把东西打印出来，而是想知道 `console.log()` *是否正常被调用*，以及*调用的参数是什么*。

- 新建 `src/test/shared/dog-test.js` 文件，加入以下内容：

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

这里我们使用了 Sinon 的 *stubs* 功能和一个 Chai 的插件。

- 运行 `yarn add --dev sinon sinon-chai` 安装需要的库文件

那么这里到底做了什么呢？首先，我们调用 `chai.use(sinonChai)` 来激活 Chai 插件。所有的奥秘都在发生在 `it()` 语句：`stub(console, 'log')`  将会为 `console.log` 打一个 stab 并监听它。当 `new Dog('Test Toby').barkInConsole()` 执行的时候，不出意外 `console.log` 应当会被调用。我们使用 `console.log.should.have.been.calledWith()` 来测试是否调用了 `console.log`。测试结束后恢复 `console.log`，使它再次正常工作。

**重要说明**：不推荐对 `console.log` 使用 stab，因为一旦测试失败，`console.log.restore()` 就不会被调用了，所以在其他测试代码执行的时候 `console.log` 一直是坏的！甚至不会打印导致测试失败的错误消息，这会很麻烦，一旦出了问题我们也不知道是什么问题。这个例子只用于说明 stub 功能。

如果一切顺利，目前应该有两个 “pass” 的测试用例了。

下一章：[12 - 使用 Flow 进行类型检查](/tutorial/12-flow)

回到[上一章](/tutorial/10-immutable-redux-improvements)或[目录](https://github.com/pd4d10/js-stack-from-scratch)
