# 5 - ES6 模块系统

将 `const Dog = require('./dog')` 替换为 `import Dog from './dog'`，这是 ES6 模块系统的语法（前一种属于 CommonJS 模块语法）。

将 `dog.js` 中的  `module.exports = Dog` 也替换为 `export default Dog`。

在 `dog.js` 中，`Dog` 这个变量只在 `export` 中使用了。因此我们可以直接导出一个匿名的类，如下：

```javascript
export default class {
  constructor(name) {
    this.name = name;
  }

  bark() {
    return `Wah wah, I am ${this.name}`;
  }
}
```

你可能已经猜到了，使用 `import`  导入 `index.js` 不一定要使用 Dog 来指代它，其他名字也可以，它只是一个代号而已。下面的代码也能正常运行：

```javascript
import Cat from './dog';

const toby = new Cat('Toby');
```
大多数时候，我们会使用与 类/模块 相同的名字。在 Gulp 文件中我们使用 `const babel = require('gulp-babel')`，这是一个反例。

那 `gulpfile.js` 中的那些 `require()` 可以替换成 `import` 吗？最新版本的 Node 支持大部分的 ES6 功能，但不支持 ES6 模块语法。不过幸运的是，可以让 Babel 来帮助 Gulp 做这件事。如果我们将 `gulpfile.js` 重命名为 `gulpfile.babel.js`，Babel 会将 `import` 导入的模块传给 Gulp。

- 将 `gulpfile.js` 重命名为 `gulpfile.babel.js`
- 替换其中的 `require()`：

```javascript
import gulp from 'gulp';
import babel from 'gulp-babel';
import { exec } from 'child_process';
```

注意 `exec` 方法直接从 `child_process` 模块中提取出来了（使用了 ES6 的解构赋值语法）。非常优雅！

- `yarn start` 还是会打印 "Wah wah, I am Toby"

下一章：[6 - 代码检查工具 ESLint](/tutorial/6-eslint)

回到[上一章](/tutorial/4-es6-syntax-class)或[目录](https://github.com/verekia/js-stack-from-scratch).
