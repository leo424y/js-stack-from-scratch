# 6 - 代码检查工具 ESLint

我们需要检查代码来发现潜在的问题。ESLint 是 ES6 代码检查的首选。在这个例子中，我们不自己配置规则，而是使用 Airbnb 的规则。它依赖了一些插件，首先需要安装它们：

- 运行 `yarn add --dev eslint eslint-config-airbnb eslint-plugin-import eslint-plugin-jsx-a11y@2.2.3 eslint-plugin-react`

一条命令里可以安装多个包。这些包都会自动添加到 `package.json` 中。

在 `package.json` 中添加 `eslintConfig` 字段：

```json
"eslintConfig": {
  "extends": "airbnb",
  "plugins": [
    "import"
  ]
},
```

`plugin` 字段用于告诉 ESLint 我们使用了 ES6 中的模块语法。

**注意**：也可以使用根目录下的 `.eslintrc.js`，`.eslintrc.json` 或 `. eslintrc.yaml` 来代替 `package.json` 中的 `eslintConfig` 字段。跟 Babel 配置类似，我们不希望根目录下有太多文件，但如果你的 ESLint 配置比较复杂，那可以考虑把它单独放在一个文件里。

我们将创建一个 Gulp 任务，用于运行 ESLint。首先需要安装 ESLint 的 Gulp 插件：

- 运行 `yarn add --dev gulp-eslint`

将以下任务添加到 `gulpfile.babel.js` 中：

```javascript
import eslint from 'gulp-eslint';

const paths = {
  allSrcJs: 'src/**/*.js',
  gulpFile: 'gulpfile.babel.js',
  libDir: 'lib',
};

// [...]

gulp.task('lint', () => {
  return gulp.src([
    paths.allSrcJs,
    paths.gulpFile,
  ])
    .pipe(eslint())
    .pipe(eslint.format())
    .pipe(eslint.failAfterError());
});
```

在这个任务中，将 `gulpfile.babel.js` 通过 `src` 包括进来了。

为 `build` 任务添加一个依赖 `lint`：

```javascript
gulp.task('build', ['lint', 'clean'], () => {
  // ...
});
```

- 运行 `yarn start`，此时会报一堆代码检查错误，以及一个关于 `console.log()` 的警告。

其中一个错误是 `'gulp' should be listed in the project's dependencies, not devDependencies (import/no-extraneous-dependencies)`。这其实不是我们想要的，因为 ESLint 不知道哪些文件是构建过程的一部分，哪些不是，所以我们需要写一些注释来告诉他。在 `gulpfile.babel.js` 的顶部添加：

```javascript
/* eslint-disable import/no-extraneous-dependencies */
```

这样 ESLint 就不会对这个文件使用 `import/no-extraneous-dependencies` 规则了。

还有一个错误：`Unexpected block statement surrounding arrow body (arrow-body-style)`。ESLint 告诉我们有更好的写法：

```javascript
() => {
  return 1;
}
```

应该写成：

```javascript
() => 1
```

在 ES6 中，如果函数体中只包含 return 语句时，可以去掉大括号，return 语句和分号。

相应的我们可以更新 Gulp 文件了：

```javascript
gulp.task('lint', () =>
  gulp.src([
    paths.allSrcJs,
    paths.gulpFile,
  ])
    .pipe(eslint())
    .pipe(eslint.format())
    .pipe(eslint.failAfterError())
);

gulp.task('clean', () => del(paths.libDir));

gulp.task('build', ['lint', 'clean'], () =>
  gulp.src(paths.allSrcJs)
    .pipe(babel())
    .pipe(gulp.dest(paths.libDir))
);
```

最后一个问题是关于 `console.log()` 的。 `console.log()` 是我们预期的结果，应该告诉 ESLint 不要检查这个规则。你可能已经猜到了，与之前类似，将 `/* eslint-disable no-console */` 添加到 `index.js` 即可。

- 运行 `yarn start`，现在应该没有任何错误了。

**注意**：本章我们学习了如何在终端中配置 ESLint。在代码构建期间，提交之前就发现潜在的错误是一个很好的功能，你可能希望将这个功能集成到 IDE 里。**不要**使用 IDE 默认的 ESLint 配置，你需要额外配置一下让 IDE 使用 `node_modules` 下的 ESLint 命令（一般是 `node_modules/.bin/eslint`）。这样才会使用我们自己定义的配置。

下一章：[7 - 前端打包工具 Webpack](/tutorial/7-client-webpack)

回到[上一章](/tutorial/5-es6-modules-syntax)或[目录](https://github.com/pd4d10/js-stack-from-scratch#目录)
