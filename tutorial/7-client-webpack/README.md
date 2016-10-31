# 7 - 前端打包工具 Webpack

## 应用程序结构

- 在根目录下新建 `dist` 目录，在其中新建一个 `index.html` 文件：

```html
<!doctype html>
<html>
  <head>
  </head>
  <body>
    <div class="app"></div>
    <script src="client-bundle.js"></script>
  </body>
</html>
```

在 `src` 文件夹中，新建 `server`，`shared`，`client` 文件夹，将 `index.js` 移动到 `server` 文件夹中，将 `dog.js` 移动到 `shared` 文件夹中。在 `client` 文件夹中新建 `app.js`。

目前我们不会涉及 Node 后端相关的知识，但将这些文件分开到不同的文件夹，有助于理解这些文件的作用。由于目录结构变了，所以我们需要将 `server/index.js` 中的 `import Dog from './dog';` 修改成 `import Dog from '../shared/dog';`，不然 ESLint 会报一个模块无法解析的错误。

在 `client/app.js` 中添加：

```javascript
import Dog from '../shared/dog';

const browserToby = new Dog('Browser Toby');

document.querySelector('.app').innerText = browserToby.bark();
```

在 `package.json` 中的 `eslintConfig` 中添加：

```json
"env": {
  "browser": true
}
```
这样可以让 ESLint 知道目前在浏览器环境中，所以 `window` 或 `document` 之类的变量一定是存在的，就不会报变量未声明的错误了。

如果你希望在前端代码中使用一些最新的 ES 功能，比如 `Promise`，需要引入 [Babel Polyfill](https://babeljs.io/docs/usage/polyfill/)。

- 运行 `yarn add babel-polyfill`

在 `app.js` 的最前面加入：

```javascript
import 'babel-polyfill';
```

这样最终打包生成的文件体积会更大，所以如果你不需要这里面的任何功能，不要这么做。为了提供固定的样板，我将把它一起打包进去，下一章的代码示例里会包括它。

## Webpack

在 Node 环境中，你可以使用 `import` 导入任何的文件，Node 根据路径自动寻找这些文件。而在浏览器端，并没有一个文件系统，所以没有可以用 `import` 导入的东西。为了让入口文件 `app.js` 知道它需要导入什么，我们要将整个依赖树“打包”到一个文件中。Webpack 就是用来做这个的。

和 Gulp 类似，Webpack 也需要一个配置文件，叫 `webpack.config.js`。为了使用 ES6 模块语法，跟之前的 Gulp 一样，需要将配置文件命名为 `webpack.config.babel.js` 即可。

- 新建 `webpack.config.babel.js` 文件
- 添加 `webpack.config.babel.js` 到 Gulp 的 `lint` 任务中，同时在 `paths` 中添加一些常量：

```javascript
const paths = {
  allSrcJs: 'src/**/*.js',
  gulpFile: 'gulpfile.babel.js',
  webpackFile: 'webpack.config.babel.js',
  libDir: 'lib',
  distDir: 'dist',
};

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
);
```

我们需要告诉 Webpack 如何处理 ES6 文件（就跟之前 Gulp 使用 `gulp-babel` 一样）。在 Webpack 中，如果需要处理除旧版本的 JavaScript 之外的文件，需要使用 *loaders*。所以我们要安装 Babel 的 loader：

- 运行 `yarn add --dev babel-loader`
- 在 `webpack.config.babel.js` 中添加以下内容：

```javascript
export default {
  output: {
    filename: 'client-bundle.js',
  },
  devtool: 'source-map',
  module: {
    loaders: [
      {
        test: /\.jsx?$/,
        loader: 'babel-loader',
        exclude: [/node_modules/],
      },
    ],
  },
  resolve: {
    extensions: ['', '.js', '.jsx'],
  },
};
```

我们一起分析一下这个文件到底做了什么：

首先 `export` 了一个对象。`output.filename` 指定了打包后的文件名。`devtool: 'source-map'` 将开启 source map，让浏览器调试更方便。`module.loaders` 中的 `test` 字段是一个 JavaScript 的正则表达式，匹配的文件将会使用 `babel-loader` 处理。在下一章中我们需要匹配 `.js` 和 `.jsx`（React），所以使用了 `/\.jsx?$/` 这个正则。`node_modules` 中的文件不需要编译，所以包含在 `exclude` 字段中，这样 Babel 不会去尝试编译这些文件，能减少构建时间。`resolve` 字段告诉 Webpack 哪些文件在 `import` 的时候可以省略扩展名，这样 `import Foo from './foo'` 中的 `foo` 可以代表 `foo.js` 或 `foo.jsx`。

配置完毕，现在我们需要运行 Webpack。

## 将 Webpack 集成到 Gulp

Webpack 可以做很多事情。如果你的项目中主要是客户端的业务逻辑，它实际上可以完全替代 Gulp。Gulp 是一个更通用的工具，它更适合用于代码检查，测试和后端任务等，同时对于新手来说比复杂的 Webpack 更容易理解。我们已经有一个健壮的 Gulp 工作流了，所以将 Webpack 集成到 Gulp 中会比较容易。

接下来我们要创建一个 Gulp 任务用于执行 Webpack。打开 `gulpfile.babel.js`。

现在不需要 `main` 任务执行 `node lib/` 命令了，取而代之的是打开 `index.html` 来运行 APP。

- 移除 `import { exec } from 'child_process'`

与 Gulp 插件类似，`webpack-stream` 这个包使我们能够很容易将 Webpack 集成到 Gulp 中。

- 安装包：`yarn add --dev webpack-stream`
- 加入以下内容：

```javascript
import webpack from 'webpack-stream';
import webpackConfig from './webpack.config.babel';
```

第二行代码导入了我们的配置文件。

如我之前所说，在下一章中我们将使用 `.jsx` 文件（在客户端，甚至是服务端），所以让我们现在先设置它。

- 将常量改成以下内容：

```javascript
const paths = {
  allSrcJs: 'src/**/*.js?(x)',
  serverSrcJs: 'src/server/**/*.js?(x)',
  sharedSrcJs: 'src/shared/**/*.js?(x)',
  clientEntryPoint: 'src/client/app.js',
  gulpFile: 'gulpfile.babel.js',
  webpackFile: 'webpack.config.babel.js',
  libDir: 'lib',
  distDir: 'dist',
};
```

`.js?(x)` 只是一个匹配 `.js` 或 `.jsx` 文件的模式。

我们现在有了我们应用程序的不同部分的常量，以及一个入口文件。

- 修改 `main` 任务如下：

```javascript
gulp.task('main', ['lint', 'clean'], () =>
  gulp.src(paths.clientEntryPoint)
    .pipe(webpack(webpackConfig))
    .pipe(gulp.dest(paths.distDir))
);
```

**注意**：`build` 任务目前会将 `src` 下的所有 `.js` 文件的 ES6 语法转换成 ES5。现在我们已经将代码分成了服务端，共享的和客户端三部分，分别位于 `server`， `shared` 和 `client` 目录下，我们可以让这个任务只编译服务端和共享的（因为客户端代码由 Webpack 负责）。但是，在测试那一章中，我们需要用 Gulp 编译客户端的代码以便在 Webpack 外部进行测试。所以，在那一章之前，这部分的构建过程并没有意义。实际上，在那之前，甚至可以不再使用 `build` 任务和 `lib` 文件夹，因为我们现在关心的是客户端打包。

- 运行 `yarn start`，文件夹下多出了一个 `client-bundle.js` 文件，在浏览器中打开 `index.html` ，将展示 "Wah wah, I am Browser Toby"。

最后一点：与 `lib` 文件夹不同， `dist/client-bundle.js` 和 `dist/client-bundle.js.map` 文件在每次构建之前都不会被 `clean` 任务清理。

- 添加 `clientBundle: 'dist/client-bundle.js?(.map)'` 到 `paths`，同时调整 `clean` 任务：

```javascript
gulp.task('clean', () => del([
  paths.libDir,
  paths.clientBundle,
]));
```

- 将 `/dist/client-bundle.js*` 添加到 `.gitignore` 文件。

下一章：[8 - React](/tutorial/8-react)

返回[上一节](/tutorial/6-eslint)或[目录](https://github.com/pd4d10/js-stack-from-scratch)
