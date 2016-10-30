# 12 - Flow

[Flow](https://flowtype.org/) 是一个静态类型检查器。它检查代码中的类型问题，可以通过注解（annotation）添加显式类型声明。

- 为了让 Babel 转换的时候移除 Flow 的注解，请运行 `yarn add --dev babel-preset-flow` 安装 Babel 的 Flow preset。然后在 `package.json` 中的 `babel.presets` 下添加 `"flow"`。
- 在根目录新建 `.flowconfig` 文件
- 运行 `yarn add --dev gulp-flowtype` 安装 Flow 的 Gulp 插件，在 `lint` 任务中添加 `flow()`，如下：

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

`abort` 选项用于在检查到错误时中断 Gulp 任务。

现在应该可以运行 Flow 了。

- 在 `src/shared/dog.js` 中添加 Flow 注解如下：

```javascript
// @flow

class Dog {
  name: string;

  constructor(name: string) {
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

`// @flow` 注释告诉 Flow 这个文件需要进行类型检查。Flow 注解通常是在函数参数或函数名后加一个冒号。如果需要进一步了解，可以查看文档。

现在运行 `yarn start`，Flow 是正常的，但 ESLint 将会提示代码的语法有错。因为我们装了 `babel-preset-flow`，所以 Babel 能正常地解析 Flow 语法，而 ESLint 不能，如果 ESLint 能通过 Babel 的解析器来工作就好了。没问题，只要安装 `babel-eslint` 这个包就好了：

- 运行 `yarn add --dev babel-eslint`

- 在 `package.json` 的 `eslintConfig` 字段添加 `"parser": "babel-eslint"`

`yarn start`，现在代码检查和类型检查应该都正常了。

现在 ESLint 和 Babel 共享同一个代码解析器，可以通过 `eslint-plugin-flowtype` 插件来检查 Flow 注解了。

- 运行 `yarn add --dev eslint-plugin-flowtype`，并在 `package.json` 中的 `eslintConfig.plugins` 添加 `"flowtype"`，向 `eslintConfig.extends` 数组添加 `"plugin:flowtype/recommended"`

现在，如果你使用 `name:string` 作为注释，ESLint 会提示冒号后少了一个空格。

**注意**：在 `package.json` 中我们添加的 `"parser": "babel-eslint"` 属性包含了 `"plugin:flowtype/recommended"` 配置，所以实际上你可以移除它，让 `package.json` 更简洁。保留它则让语义显得更明确，这取决于你的个人喜好。本教程旨在最简洁的配置文件，所以我将它删除了。

- 现在可以在 `src` 目录下的每个 `.js` 和 `.jsx` 文件中添加 `// @flow` 注释了，运行 `yarn test` 或 `yarn start`， 在每个 Flow 提示你的地方添加注解。

在 `src/client/component/message.jsx` 文件中，有一种比较违反直觉的情况如下：

```javascript
const Message = ({ message }: { message: string }) => <div>{message}</div>;
```

在解构函数参数时，必须使用对象字面量来添加注解。

另一个可能会遇到的情况是，在 `src/client/reducers/dog-reducer.js` 中，Flow 会提示 Immutable 没有 export default。这个问题在 [#863 on Immutable](https://github.com/facebook/immutable-js/issues/863) 中提出了两个解决方法：

```javascript
import { Map as ImmutableMap } from 'immutable';
// or
import * as Immutable from 'immutable';
```

在 Immutable 正式解决这个问题之前，选择你喜欢的方式吧。我个人倾向于使用 `import * as Immutable from 'immutable'`，因为它更短，而且这个问题修复以后不需要更改也能正常运行。

**注意**：如果 Flow 在 `node_modules` 文件夹中检测到类型错误，请在 `.flowconfig` 中添加一个`[ignore]` 来忽略特定的包（不要忽略整个 `node_modules` 目录）。看起来像这样

```
[ignore]

.*/node_modules/gulp-flowtype/.*
```
在我这的情况是，Atom 编辑器的 `linter-flow` 插件会检测 `node_modules/gulp-flowtype` 目录中的类型错误，针对那些用 `// @flow` 注释过的文件。

现在，你的代码已经通过了代码检查，类型检查和测试的考验了，好样的！

回到[上一章](/tutorial/11-testing-mocha-chai-sinon)或[目录](https://github.com/pd4d10/js-stack-from-scratch)
