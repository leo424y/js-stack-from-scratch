# 1 - Node，NPM，Yarn 和 package.json

在本節中，我們將學習如何設定 Node，NPM，Yarn 和 `package.json`。

首先需要安裝 Node，它提供了後端 JavaScript 的執行環境，同時還包括構建前端技術棧所需的所有工具。

macOS 或 Windows 使用者可以直接[下載安裝檔案](https://nodejs.org/en/download/current/)，Linux 使用者可以通過[包管理器安裝](https://nodejs.org/en/download/package-manager/)。

例如，在 **Ubuntu / Debian** 上，您可以執行以下命令來安裝 Node：

```bash
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs
```

你需要大於 6.5.0 的 Node 版本。

`npm` 是 Node 的預設包管理器，不需要手動安裝。

**注意**：如果你之前已經安裝過 Node，可以使用 `nvm` ([Node Version Manager](https://github.com/creationix/nvm)) 安裝最新版本的 Node。

[Yarn](https://yarnpkg.com/) 是另一個包管理器，它比 NPM 快許多，而且能離線快取，在包的依賴管理上[更可靠](https://yarnpkg.com/en/docs/yarn-lock)。Yarn 於 2016 年 10 月 [釋出](https://code.facebook.com/posts/1840075619545360) 以來就獲得了廣泛的使用，正在成為 JavaScript 社羣選擇的新的包管理器。我們將在本教程中使用 Yarn。如果你想使用 NPM，用 `npm install --save` 和 `npm install --save-dev` 分別替換 `yarn add` 和 `yarn add —dev` 命令即可。

- 按照[這個說明](https://yarnpkg.com/en/docs/install)安裝 Yarn。你可以使用 `npm install -g yarn` 或 `sudo npm install -g yarn` 安裝它（是的，我們可以使用NPM來安裝Yarn，就像使用 Internet Explorer 或 Safari 安裝Chrome 一樣！）。
- 建立一個新資料夾，並 `cd` 到資料夾中。
- 執行 `yarn init`，並按照提示輸入一些欄位（使用 `yarn init -y` 可以跳過輸入欄位的環節），將自動生成一個 `package.json` 檔案。
- 新建 `index.js` 檔案，內容為 `console.log('Hello world')`。
- 在當前資料夾下執行 `node .`（Node 預設會去找當前資料夾下的 `index.js`）。將列印 “Hello world”。

執行 `node .` 可能有點太容易了。我們將使用 NPM / Yarn 指令碼來觸發程式碼的執行。這樣做的好處是，即使我們的程式變得更復雜，也能使用簡單的一個命令 `yarn start` 來執行整個程式。

- 在 `package.json` 中增加 `scripts` 欄位如下：

```json
"scripts": {
  "start": "node ."
}
```

`package.json` 必須是有效的 JSON 檔案，這意味著不能使用尾逗號。手動編輯 `package.json` 檔案時要注意這一點。

- 執行 `yarn start`，將列印 `Hello world`。
- 新建一個 `.gitignore` 檔案，增加以下內容：

```gitignore
npm-debug.log
yarn-error.log
```

**注意**：你可能注意到每章的 `package.json` 檔案都有一個 `tutorial-test` 指令碼。這些指令碼是用於測試的，確保 `yarn && yarn start` 執行正確。你可以在自己的項目中刪除它們。

下一章：[2 - 包的安裝與使用](/tutorial/2-packages)

回到[目錄](https://github.com/pd4d10/js-stack-from-scratch#目錄)
