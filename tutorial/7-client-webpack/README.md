# 7 - 前端打包工具 Webpack

## 應用程式結構

- 在根目錄下新建 `dist` 目錄，在其中新建一個 `index.html` 檔案：

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

在 `src` 資料夾中，新建 `server`，`shared`，`client` 資料夾，將 `index.js` 移動到 `server` 資料夾中，將 `dog.js` 移動到 `shared` 資料夾中。在 `client` 資料夾中新建 `app.js`。

目前我們不會涉及 Node 後端相關的知識，但將這些檔案分開到不同的資料夾，有助於理解這些檔案的作用。由於目錄結構變了，所以我們需要將 `server/index.js` 中的 `import Dog from './dog';` 修改成 `import Dog from '../shared/dog';`，不然 ESLint 會報一個模組無法解析的錯誤。

在 `client/app.js` 中新增：

```javascript
import Dog from '../shared/dog';

const browserToby = new Dog('Browser Toby');

document.querySelector('.app').innerText = browserToby.bark();
```

在 `package.json` 中的 `eslintConfig` 中新增：

```json
"env": {
  "browser": true
}
```

這樣可以讓 ESLint 知道目前在瀏覽器環境中，所以 `window` 或 `document` 之類的變數一定是存在的，就不會報變數未聲明的錯誤了。

如果你希望在前端程式碼中使用一些最新的 ES 功能，比如 `Promise`，需要引入 [Babel Polyfill](https://babeljs.io/docs/usage/polyfill/)。

- 執行 `yarn add babel-polyfill`

在 `app.js` 的最前面加入：

```javascript
import 'babel-polyfill';
```

這樣最終打包生成的檔案體積會更大，所以如果你不需要這裡面的任何功能，不要這麼做。為了提供固定的樣板，我將把它一起打包進去，下一章的程式碼示例裡會包括它。

## Webpack

在 Node 環境中，你可以使用 `import` 匯入任何的檔案，Node 根據路徑自動尋找這些檔案。而在瀏覽器端，並沒有一個檔案系統，所以沒有可以用 `import` 匯入的東西。為了讓入口檔案 `app.js` 知道它需要匯入什麼，我們要將整個依賴樹“打包”到一個檔案中。Webpack 就是用來做這個的。

和 Gulp 類似，Webpack 也需要一個配置檔案，叫 `webpack.config.js`。為了使用 ES6 模組語法，跟之前的 Gulp 一樣，需要將配置檔案命名為 `webpack.config.babel.js` 即可。

- 新建 `webpack.config.babel.js` 檔案
- 新增 `webpack.config.babel.js` 到 Gulp 的 `lint` 任務中，同時在 `paths` 中新增一些常量：

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

我們需要告訴 Webpack 如何處理 ES6 檔案（就跟之前 Gulp 使用 `gulp-babel` 一樣）。在 Webpack 中，如果需要處理除舊版本的 JavaScript 之外的檔案，需要使用 *loaders*。所以我們要安裝 Babel 的 loader：

- 執行 `yarn add --dev babel-loader`
- 在 `webpack.config.babel.js` 中新增以下內容：

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

我們一起分析一下這個檔案到底做了什麼：

首先 `export` 了一個物件。`output.filename` 指定了打包後的檔名。`devtool: 'source-map'` 將開啟 source map，讓瀏覽器偵錯更方便。`module.loaders` 中的 `test` 欄位是一個 JavaScript 的正規表示式，匹配的檔案將會使用 `babel-loader` 處理。在下一章中我們需要匹配 `.js` 和 `.jsx`（React），所以使用了 `/\.jsx?$/` 這個正則。`node_modules` 中的檔案不需要編譯，所以包含在 `exclude` 欄位中，這樣 Babel 不會去嘗試編譯這些檔案，能減少構建時間。`resolve` 欄位告訴 Webpack 哪些檔案在 `import` 的時候可以省略副檔名，這樣 `import Foo from './foo'` 中的 `foo` 可以代表 `foo.js` 或 `foo.jsx`。

配置完畢，現在我們需要執行 Webpack。

## 將 Webpack 整合到 Gulp

Webpack 可以做很多事情。如果你的項目中主要是客戶端的業務邏輯，它實際上可以完全替代 Gulp。Gulp 是一個更通用的工具，它更適合用於程式碼檢查，測試和後端任務等，同時對於新手來說比複雜的 Webpack 更容易理解。我們已經有一個健壯的 Gulp 工作流了，所以將 Webpack 整合到 Gulp 中會比較容易。

接下來我們要建立一個 Gulp 任務用於執行 Webpack。開啟 `gulpfile.babel.js`。

現在不需要 `main` 任務執行 `node lib/` 命令了，取而代之的是開啟 `index.html` 來執行 APP。

- 移除 `import { exec } from 'child_process'`

與 Gulp 外掛類似，`webpack-stream` 這個包使我們能夠很容易將 Webpack 整合到 Gulp 中。

- 安裝包：`yarn add --dev webpack-stream`
- 加入以下內容：

```javascript
import webpack from 'webpack-stream';
import webpackConfig from './webpack.config.babel';
```

第二行程式碼匯入了我們的配置檔案。

如我之前所說，在下一章中我們將使用 `.jsx` 檔案（在客戶端，甚至是服務端），所以讓我們現在先設定它。

- 將常量改成以下內容：

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

`.js?(x)` 只是一個匹配 `.js` 或 `.jsx` 檔案的模式。

我們現在有了我們應用程式的不同部分的常量，以及一個入口檔案。

- 修改 `main` 任務如下：

```javascript
gulp.task('main', ['lint', 'clean'], () =>
  gulp.src(paths.clientEntryPoint)
    .pipe(webpack(webpackConfig))
    .pipe(gulp.dest(paths.distDir))
);
```

**注意**：`build` 任務目前會將 `src` 下的所有 `.js` 檔案的 ES6 語法轉換成 ES5。現在我們已經將程式碼分成了服務端，共享的和客戶端三部分，分別位於 `server`， `shared` 和 `client` 目錄下，我們可以讓這個任務只編譯服務端和共享的（因為客戶端程式碼由 Webpack 負責）。但是，在測試那一章中，我們需要用 Gulp 編譯客戶端的程式碼以便在 Webpack 外部進行測試。所以，在那一章之前，這部分的構建過程並沒有意義。實際上，在那之前，甚至可以不再使用 `build` 任務和 `lib` 資料夾，因為我們現在關心的是客戶端打包。

- 執行 `yarn start`，資料夾下多出了一個 `client-bundle.js` 檔案，在瀏覽器中開啟 `index.html` ，將展示 "Wah wah, I am Browser Toby"。

最後一點：與 `lib` 資料夾不同， `dist/client-bundle.js` 和 `dist/client-bundle.js.map` 檔案在每次構建之前都不會被 `clean` 任務清理。

- 新增 `clientBundle: 'dist/client-bundle.js?(.map)'` 到 `paths`，同時調整 `clean` 任務：

```javascript
gulp.task('clean', () => del([
  paths.libDir,
  paths.clientBundle,
]));
```

- 將 `/dist/client-bundle.js*` 新增到 `.gitignore` 檔案。

下一章：[8 - React](/tutorial/8-react)

返回[上一節](/tutorial/6-eslint)或[目錄](https://github.com/pd4d10/js-stack-from-scratch#目錄)
