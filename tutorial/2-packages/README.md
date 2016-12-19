# 2 - 安装和使用包

在本节中我们将学习安装和使用包。每个“包”都是别人编写的代码片段的集合，你可以在你自己的代码中使用。它可以是任何代码。下面我们将尝试使用一个帮助你操作颜色的包。

- 运行 `yarn add color`，安装名为 `color` 的包

打开 `package.json`，可以看到 `color` 已经被自动加入到 `dependencies` 中了。

目录中多了一个 `node_modules` 文件夹，用于存储包。

- 添加 `node_modules/` 到 `.gitignore` 文件中（如果当前目录还不是一个 git 仓库，执行 `git init` 创建一个新的 git 仓库）。

Yarn 生成了一个 `yarn.lock` 文件，它应该被提交到 git 仓库，确保团队中的每个人都使用相同的版本。如果你使用的是 NPM，那么应该使用 *shrinkwrap*。

- 在 `index.js` 中增加 `const Color = require('color');`
- 可以这样来使用它：`const redHexa = Color({r: 255, g: 0, b: 0}).hex();`
- 增加 `console.log(redHexa)`。
- 运行 `yarn start` 应该显示 `#FF0000`。

恭喜，你已经学会安装和使用包了！

`color` 只是用于教你学会如何使用包。如果不再需要这个包了，可以卸载它：

- 运行 `yarn remove color`

**注意**：有两种包依赖关系， `"dependencies"` 和 `"devDependencies"`。`"dependencies"` 更通用，而 `"devDependencies"` 是只在开发阶段使用的包（比如构建工具，代码检查工具等）。执行 `yarn add --dev [package]` 可以把包添加到 `"devDependencies"` 中。

下一章：[3 - 使用 Babel 和 Gulp 配置 ES6 开发环境](/tutorial/3-es6-babel-gulp)

回到[上一章](/tutorial/1-node-npm-yarn-package-json)或[目录](https://github.com/pd4d10/js-stack-from-scratch#目录)
