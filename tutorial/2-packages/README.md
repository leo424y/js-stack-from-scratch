# 2 - 安裝和使用包

在本節中我們將學習安裝和使用包。每個“包”都是別人編寫的程式碼片段的集合，你可以在你自己的程式碼中使用。它可以是任何程式碼。下面我們將嘗試使用一個幫助你操作顏色的包。

- 執行 `yarn add color`，安裝名為 `color` 的包

開啟 `package.json`，可以看到 `color` 已經被自動加入到 `dependencies` 中了。

目錄中多了一個 `node_modules` 資料夾，用於儲存包。

- 新增 `node_modules/` 到 `.gitignore` 檔案中（如果當前目錄還不是一個 git 倉庫，執行 `git init` 建立一個新的 git 倉庫）。

Yarn 生成了一個 `yarn.lock` 檔案，它應該被提交到 git 倉庫，確保團隊中的每個人都使用相同的版本。如果你使用的是 NPM，那麼應該使用 *shrinkwrap*。

- 在 `index.js` 中增加 `const Color = require('color');`
- 可以這樣來使用它：`const redHexa = Color({r: 255, g: 0, b: 0}).hex();`
- 增加 `console.log(redHexa)`。
- 執行 `yarn start` 應該顯示 `#FF0000`。

恭喜，你已經學會安裝和使用包了！

`color` 只是用於教你學會如何使用包。如果不再需要這個包了，可以解除安裝它：

- 執行 `yarn remove color`

**注意**：有兩種包依賴關係， `"dependencies"` 和 `"devDependencies"`。`"dependencies"` 更通用，而 `"devDependencies"` 是隻在開發階段使用的包（比如構建工具，程式碼檢查工具等）。執行 `yarn add --dev [package]` 可以把包新增到 `"devDependencies"` 中。

下一章：[3 - 使用 Babel 和 Gulp 配置 ES6 開發環境](/tutorial/3-es6-babel-gulp)

回到[上一章](/tutorial/1-node-npm-yarn-package-json)或[目錄](https://github.com/pd4d10/js-stack-from-scratch#目錄)
