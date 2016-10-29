# 4 - 使用 ES6 中的 class

- 新建一个文件 `src/dog.js`，包含以下内容：

```javascript
class Dog {
  constructor(name) {
    this.name = name;
  }

  bark() {
    return `Wah wah, I am ${this.name}`;
  }
}

module.exports = Dog;
```

如果你之前接触过面向对象编程，你应该不会感到陌生。对于 JavaScript 来说，这是最近才有的语法。这个类通过赋值给 `module.exports` 暴露出来。

ES6 代码中经常使用的有类，`const` 和 `let`，模板字符串和箭头函数（`(param) => { console.log('Hi'); }`），尽管我们在这个例子中没有使用到上面说的这些。

在 `src/index.js` 加入如下内容：

```javascript
const Dog = require('./dog');

const toby = new Dog('Toby');

console.log(toby.bark());
```
你可能注意到了，跟引用第三方包 `color ` 的方式有点不一样，当我们需要引用本地的文件时，在 `require()` 中使用了 `./`。

- 运行 `yarn start` ，将打印  'Wah wah, I am Toby'。
- 看一下 `lib` 中编译后的文件长什么样（`const` 都被 `var` 替换了）。


下一章：[5 - ES6 模块系统](/tutorial/5-es6-modules-syntax)

回到[上一章](/tutorial/3-es6-babel-gulp)或[目录](https://github.com/pd4d10/js-stack-from-scratch).
