# 3 - 使用 Babel 和 Gulp 搭建 ES6 开发环境

我们将使用 ES6 语法，它相比于 ES5 有了很大改进。到目前为止，几乎所有的浏览器和 JS 运行环境都支持 ES5，但不是所有的都支持 ES6。所以我们将使用一个叫做 Babel 的工具，将 ES6 代码转换成 ES5 代码。为了运行 Babel，我们需要 Gulp。它类似于 `package.json` 的 `scripts` 中的任务，但是将任务写在 JS 文件中比使用 JSON 文件更简单清晰。我们将安装 Gulp 和 Gulp 的 Babel 插件：

- 运行 `yarn add --dev gulp`
- 运行 `yarn add --dev gulp-babel`
- 运行 `yarn add --dev babel-preset-latest`
- 运行 `yarn add --dev del`（执行 `clean` 任务使用的包）
- 在 `package.json` 中增加一个 `babel` 字段，添加如下代码将使用最新的 Babel 配置：

```json
"babel": {
  "presets": [
    "latest"
  ]
},
```

**注意**：你也可以使用根目录下的 `.babelrc` 文件来保存 Babel 配置。随着项目变得复杂，根目录的文件将越来越多，所以我们在 `package.json` 中保存 Babel 的配置。

- 将 `index.js` 移动到一个名为 `src` 的新文件夹，用于存放 ES6 代码，编译后的 ES5 代码存放在 `lib` 文件夹。Gulp 和 Babel 会帮我们完成编译。删除  `index.js` 中有关 `color` 的代码，将其替换为以下更简单的版本：

```javascript
const str = 'ES6';
console.log(`Hello ${str}`);
```

这里用到了*模板字符串*，这是一个 ES6 中的新功能，可以直接使用 `${}` 在字符串中插入变量。有一点需要注意，模板字符串使用的是反引号。

- 新建 `gulpfile.js` 文件，包含以下内容：

```javascript
const gulp = require('gulp');
const babel = require('gulp-babel');
const del = require('del');
const exec = require('child_process').exec;

const paths = {
  allSrcJs: 'src/**/*.js',
  libDir: 'lib',
};

gulp.task('clean', () => {
  return del(paths.libDir);
});

gulp.task('build', ['clean'], () => {
  return gulp.src(paths.allSrcJs)
    .pipe(babel())
    .pipe(gulp.dest(paths.libDir));
});

gulp.task('main', ['build'], (callback) => {
  exec(`node ${paths.libDir}`, (error, stdout) => {
    console.log(stdout);
    return callback(error);
  });
});

gulp.task('watch', () => {
  gulp.watch(paths.allSrcJs, ['main']);
});

gulp.task('default', ['watch', 'main']);
```

让我们花点时间来理解这一大段代码。

Gulp 本身的 API 很简单。使用 `gulp.task` 定义一系列的任务，使用 `gulp.src` 读取文件，使用 `.pipe()`（这里是 `babel()`）进行一系列的处理，并将使用 `gulp.dest` 输出处理后的文件。可以使用 `gulp.watch` 监听文件的更改。 将一个数组（如 `['build']`）作为第二个参数传递给 `gulp.task`，可以定义这个任务的依赖，在执行这个任务之前先执行他依赖的任务。更详细的介绍请参阅[Gulp 的文档](https://github.com/gulpjs/gulp)。

首先，定义一个 `paths` 对象来存储我们需要文件路径，这样可以并保持 DRY（不要重复自身）。

然后，定义 5 个任务： `build`， `clean`， `main`， `watch` 和  `default`。

- `build` 任务读取 `src` 下的所有文件，使用 Babel 转换，将转换后的文件写入到 `lib` 下。
- `clean` 任务用于在每次执行 `build` 之前删除 `lib` 文件夹下的所有内容。这是一个很有用的任务。当你重命名或删除了 `src` 下的一些文件，这个任务可以帮助你清除旧的编译文件。如果你不小心构建失败了，这能确保 `lib` 文件夹与 `src` 文件夹下的内容保持同步，即使你并不知道构建失败了。使用 `del` 删除文件，它遵循 Gulp 的流式处理方式（这是Gulp [推荐](https://github.com/gulpjs/gulp/blob/master/docs/recipes/delete-files-folder.md)的方式）。
- `main` 任务与之前章节里提到的  `node .` 是等价的。不过这次我们希望执行 `lib/index.js`。我们已经知道 Node 默认会去找 `index.js` 文件，所以运行 `node lib` 即可（在这里我们定义了一个变量 `libDir`，来保持 DRY）。`require('child_process').exec` 和 `exec` 是 Node 中的原生方法，用于执行 shell 命令。将 `stdout` 的内容转发到 `console.log()` 中，如果报错了，使用 `gulp.task` 的回调函数返回错误。你可能觉得不能彻底理解这部分内容，不用担心，只需要记住一点， 这个任务相当于执行了 `node lib`。
- `watch` 任务监听文件改动，当有文件内容发生变化时，执行 `main` 任务。
- 如果你直接从命令行调用 `gulp`，将默认运行 `default` 任务。在这个例子中，我们希望它运行 `watch` 和 `main`（首次执行）。

**注意**：你可能会感到奇怪，这个 Gulp 文件中的代码使用了一些 ES6 语法，我们并没有用 Babel 转换它，但它并没有报错。这是因为我们使用的 Node 版本支持某些 ES6 功能（确保你的 Node 版本大于 6.5.0，版本可以通过 `node -v` 查看）。

好！我们做完了。看下程序是不是能跑起来。

- 在 `package.json` 中，将 `start` 脚本改成 `"start": "gulp"`。
- 运行 `yarn start`，将会打印 "Hello ES6" 并开始监听文件的更改。你可以试试在 `src/index.js` 中写一些有语法错误的代码，看下 Gulp 是不是报错了。
- 添加 `/lib/` 到 `.gitignore` 文件中

下一章：[4 - 使用 ES6 中的 class](/tutorial/4-es6-syntax-class)

回到[上一章](/tutorial/2-packages)或[目录](https://github.com/pd4d10/js-stack-from-scratch#目录)
