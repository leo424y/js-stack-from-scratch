# 6 - 程式碼檢查工具 ESLint

我們需要檢查程式碼來發現潛在的問題。ESLint 是 ES6 程式碼檢查的首選。在這個例子中，我們不自己配置規則，而是使用 Airbnb 的規則。它依賴了一些外掛，首先需要安裝它們：

- 執行 `yarn add --dev eslint eslint-config-airbnb eslint-plugin-import eslint-plugin-jsx-a11y@2.2.3 eslint-plugin-react`

一條命令裡可以安裝多個包。這些包都會自動新增到 `package.json` 中。

在 `package.json` 中新增 `eslintConfig` 欄位：

```json
"eslintConfig": {
  "extends": "airbnb",
  "plugins": [
    "import"
  ]
},
```

`plugin` 欄位用於告訴 ESLint 我們使用了 ES6 中的模組語法。

**注意**：也可以使用根目錄下的 `.eslintrc.js`，`.eslintrc.json` 或 `. eslintrc.yaml` 來代替 `package.json` 中的 `eslintConfig` 欄位。跟 Babel 配置類似，我們不希望根目錄下有太多檔案，但如果你的 ESLint 配置比較複雜，那可以考慮把它單獨放在一個檔案裡。

我們將建立一個 Gulp 任務，用於執行 ESLint。首先需要安裝 ESLint 的 Gulp 外掛：

- 執行 `yarn add --dev gulp-eslint`

將以下任務新增到 `gulpfile.babel.js` 中：

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

在這個任務中，將 `gulpfile.babel.js` 通過 `src` 包括進來了。

為 `build` 任務新增一個依賴 `lint`：

```javascript
gulp.task('build', ['lint', 'clean'], () => {
  // ...
});
```

- 執行 `yarn start`，此時會報一堆程式碼檢查錯誤，以及一個關於 `console.log()` 的警告。

其中一個錯誤是 `'gulp' should be listed in the project's dependencies, not devDependencies (import/no-extraneous-dependencies)`。這其實不是我們想要的，因為 ESLint 不知道哪些檔案是構建過程的一部分，哪些不是，所以我們需要寫一些註釋來告訴他。在 `gulpfile.babel.js` 的頂部新增：

```javascript
/* eslint-disable import/no-extraneous-dependencies */
```

這樣 ESLint 就不會對這個檔案使用 `import/no-extraneous-dependencies` 規則了。

還有一個錯誤：`Unexpected block statement surrounding arrow body (arrow-body-style)`。ESLint 告訴我們有更好的寫法：

```javascript
() => {
  return 1;
}
```

應該寫成：

```javascript
() => 1
```

在 ES6 中，如果函數體中只包含 return 語句時，可以去掉大括號，return 語句和分號。

相應的我們可以更新 Gulp 檔案了：

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

最後一個問題是關於 `console.log()` 的。 `console.log()` 是我們預期的結果，應該告訴 ESLint 不要檢查這個規則。你可能已經猜到了，與之前類似，將 `/* eslint-disable no-console */` 新增到 `index.js` 即可。

- 執行 `yarn start`，現在應該沒有任何錯誤了。

**注意**：本章我們學習瞭如何在終端中配置 ESLint。在程式碼構建期間，提交之前就發現潛在的錯誤是一個很好的功能，你可能希望將這個功能整合到 IDE 裡。**不要**使用 IDE 預設的 ESLint 配置，你需要額外配置一下讓 IDE 使用 `node_modules` 下的 ESLint 命令（一般是 `node_modules/.bin/eslint`）。這樣才會使用我們自己定義的配置。

下一章：[7 - 前端打包工具 Webpack](/tutorial/7-client-webpack)

回到[上一章](/tutorial/5-es6-modules-syntax)或[目錄](https://github.com/pd4d10/js-stack-from-scratch#目錄)
