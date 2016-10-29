# 1 - Node，NPM，Yarn 和 package.json

在本节中，我们将学习如何设置 Node，NPM，Yarn 和 `package.json`。

首先需要安装 Node，它提供了后端 JavaScript 的运行环境，同时还包括构建前端技术栈所需的所有工具。

macOS 或 Windows 用户可以直接[下载安装文件](https://nodejs.org/en/download/current/)，Linux 用户可以通过[包管理器安装](https://nodejs.org/en/download/package-manager/)。

例如，在 **Ubuntu / Debian** 上，您可以运行以下命令来安装 Node：

```bash
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs
```
你需要大于 6.5.0 的 Node 版本。

`npm` 是 Node 的默认包管理器，不需要手动安装。

**注意**：如果你之前已经安装过 Node，可以使用 `nvm` ([Node Version Manager](https://github.com/creationix/nvm)) 安装最新版本的 Node。

[Yarn](https://yarnpkg.com/) 是另一个包管理器，它比 NPM 快许多，而且能离线缓存，在包的依赖管理上更可靠。Yarn 于 2016 年 10 月 [发布](https://code.facebook.com/posts/1840075619545360) 以来就获得了广泛的使用，正在成为 JavaScript 社区选择的新的包管理器。我们将在本教程中使用 Yarn。如果你想使用 NPM，用 `npm install --save` 和 `npm install --dev` 分别替换 `yarn add` 和 `yarn add —dev` 命令即可。

- 按照[这个说明](https://yarnpkg.com/en/docs/install)安装 Yarn。你可以使用 `npm install -g yarn` 或 `sudo npm install -g yarn` 安装它（是的，我们可以使用NPM来安装Yarn，就像使用 Internet Explorer 或 Safari 安装Chrome 一样！）。
- 创建一个新文件夹，并 `cd` 到文件夹中。
- 运行 `yarn init`，并按照提示输入一些字段（使用 `yarn init -y` 可以跳过输入字段的环节），将自动生成一个 `package.json` 文件。
- 新建 `index.js` 文件，内容为 `console.log('Hello world')`。
- 在当前文件夹下运行 `node .`（Node 默认会去找当前文件夹下的 `index.js`）。将打印 “Hello world”。

Running `node .` to execute our program is a bit too low-level. We are going to use an NPM/Yarn script to trigger the execution of that code instead. That will give us a nice abstraction to be able to always use `yarn start`, even when our program gets more complicated.

运行 `node .` 可能有点太容易了。我们将使用 NPM / Yarn 脚本来触发代码的执行。这样做的好处是，即使我们的程序变得更复杂，也能使用简单的一个命令 `yarn start` 来运行整个程序。

- 给 `package.json` 中的 `scripts` 对象增加 `"start": "node ."`

`package.json` 必须是有效的 JSON 文件，这意味着不能使用尾逗号。手动编辑 `package.json` 文件时要注意这一点。

- 运行 `yarn start`，将打印 `Hello world`。
- 新建一个 `.gitignore` 文件，增加以下内容：

```
npm-debug.log
yarn-error.log
```

**Note**: If you take a look at the `package.json` files I provide, you will see a `tutorial-test` script in every chapter. Those scripts let me test that the chapter works fine when running `yarn && yarn start`. You can delete them in your own projects.

**注意**：你可能注意到每章的 `package.json` 文件都有一个 `tutorial-test` 脚本。这些脚本是用于测试的，确保 `yarn && yarn start` 运行正确。你可以在自己的项目中删除它们。

下一章：[2 - 包的安装与使用](/tutorial/2-packages)

回到[目录](https://github.com/pd4d10/js-stack-from-scratch)

