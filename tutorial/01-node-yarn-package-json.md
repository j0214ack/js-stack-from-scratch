# 01 - Node, Yarn, 以及 `package.json`

本章的程式碼可以在 [這裡](https://github.com/verekia/js-stack-walkthrough/tree/master/01-node-yarn-package-json) 取得

本章中，我們將會把 Node, Yarn 還有基本的的 `package.json` 檔案設定好，並且試著使用一個套件

## Node

> 💡 **[Node.js](https://nodejs.org/)** 是一個 JavaScript 執行環境。通常最常使用在後端開發，但也可作為平常的腳本語言使用。而在前端開發的情境中， Node.js 會用來做許多工作像是 linting (譯注：一套檢查程式文法與風格的軟體)、測試、組裝檔案等等。

在這個教學裡，幾乎所有事都是透過 Node 完成的。請前往 [download page](https://nodejs.org/en/download/current/) 下載 **macOS** 或 **Windows** 的執行檔。如果你使用 Linux 系統請查看 [package manager installations page](https://nodejs.org/en/download/package-manager/) 。

例如：
在 **Ubuntu / Debian** ，你需要執行下列的指令來安裝 Node:

```sh
curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
sudo apt-get install -y nodejs
```

你需要版本大於 > 6.5.0 的 Node.js

## NVM

如果你已經安裝了 Node ，或者想要有更高的彈性，請安裝 NVM ([Node Version Manager](https://github.com/creationix/nvm))，透過 NVM 安裝最新版本的 Node ，並切換到該版本。

（譯注： `nvm install [version]` 安裝、 `nvm use [version]` 切換。）

## NPM

NPM 是 Node 預設的套件管理員。在你安裝 Node 的時候它就會自動安裝。套件管理員的用途是安裝並且管理套件 （套件是由你或其他人撰寫的一組一組的程式碼）。在這個教學裡，我們將會用到非常多的套件，但我們會使用 Yarn ，那是另外一個套件管理員。


## Yarn

> 💡 **[Yarn](https://yarnpkg.com/)** 是一個 Node.js 套件管理員，比 NPM 快速，支援離線使用，而且能夠 [更可預測地取得](https://yarnpkg.com/en/docs/yarn-lock) 相依套件。

自從 Yarn 在 2016 年十月 [釋出](https://code.facebook.com/posts/1840075619545360) 以來， JavaScript 社群很快地就採納了這個選項。如果你想要繼續使用 NPM ，你只要將本教學裡所有的 `yarn add` 和 `yarn add --dev` 換成 `npm install --save` 和 `npm install --save-dev` 即可。

從 [說明頁面](https://yarnpkg.com/en/docs/install) 找到你的作業系統，並依照指示安裝。如果你的系統是 macOS 或 Unix ，我建議使用在 *Alternatives* 分頁裡的 **Installation Script** 來安裝，以 [避免](https://github.com/yarnpkg/yarn/issues/1505) 需要依賴在其他套件管理員的情形發生。

```sh
curl -o- -L https://yarnpkg.com/install.sh | bash
```

## `package.json`

> 💡 **[package.json](https://yarnpkg.com/en/docs/package-json)** 是一個用來描述並且設置你的 JavaScript 專案的檔案。裡面會有一般性的資訊（你的專案名稱、版本、貢獻者、版權執照等等）、你所使用的工具的設置選項、甚至有一個部分可以設置想要執行的 *tasks*。

- 建立一個新工作資料夾，並且 `cd` 到裡面
- 執行 `yarn init` 並且回答畫面上的問題（用 `yarn init -y` 跳過所有問題），如此就可以自動產生出一個 `package.json` 檔案

以下是我將會在本教學裡使用的基本 `package.json`：

```json
{
  "name": "your-project",
  "version": "1.0.0",
  "license": "MIT"
}
```

## Hello World

- 創造一個 `index.js` 檔案，內含 `console.log('Hello world')`

🏁 在資料夾裡執行 `node .`（`index.js` 是 Node 預設在資料夾中會查看的檔案），我們應該就會看到 "Hello world" 顯示在終端機內。

**備註**：看見 🏁 賽車旗表情符號了嗎？你每達到一個 **檢查點** 時就會看見它。有時候我們會一口氣作出大量的更動，期間你所撰寫的程式碼可能會無法執行，直到達到下一個檢查點。

## `start` 腳本

使用 `node .` 來執行我們的程式，有點太低階了。我們將會改用 NPM/Yarn 腳本來觸發程式碼執行。這會給我們一個很棒的抽象層，讓我們可以用永遠用 `yarn start` 來執行程式，即使我們的程式在將來變得非常複雜。

- 在 `package.json` 裡，增加一個 `scripts` 物件：

```json
{
  "name": "your-project",
  "version": "1.0.0",
  "license": "MIT",
  "scripts": {
    "start": "node ."
  }
}
```

`start` 是我們給予的 *task* 名稱，這個 *task* 會執行我們的程式。在這個教學裡，我們將會在 `scripts` 物件製作非常多不同的 tasks 。 `start` 通常是一個應用程式的預設指令的名字。其他標準的 task 名稱像是 `stop` 和 `test`。

`package.json` 必須要是合法的 JSON 檔案，也就是說你不能夠有留在尾巴的逗號。當你手動編輯你的 `package.json` 檔案的時候請特別注意。

🏁 執行 `yarn start`。終端機應該會印出 `Hello world`。

## Git 與 `.gitignore`

- 用 `git init` 初始化你的 Git repository

- 製作一個 `.gitignore` 檔案，並且加入下列內容到檔案裡：

```gitignore
.DS_Store
/*.log
```

`.DS_Store` 檔案是 macOS 自動生成的檔案，你永遠不該把它放到你的 repository 之中。

`npm-debug.log` 和 `yarn-error.log` 是你的套件管理員遇到錯誤時會製作的檔案，我們不希望這些東西出現在我們的版本控制之中。

## Installing and using a package

在這個部分我們會安裝並且使用一個套件。一個「套件」其實就是某些人寫的一段程式碼，而你可以在你自己的程式當中使用它們。它們可以是任何東西。以下範例裡，我們將試著用一個套件，幫助你操作顏色。

- 用 `yarn add color` 安裝社群製作的 `color` 套件。

打開你的 `package.json` ，在 `dependencies` 裡查看 Yarn 自動幫你增加的 `color` 套件

你會注意到一個 `node_modules` 資料夾在你的專案資料夾裡產生了，這是拿來儲存你安裝的套件的資料夾。

- 將 `node_modules/` 加到你的 `.gitignore` 之中。

同時你也會注意到 Yarn 產生了一個 `yanr.lock` 檔案。你應該要將這個檔案加入到版本控制當中，這個檔案可以確保你的團隊都使用同樣的套件版本。如果你使用的是 NPM ，有著同樣功能的檔案叫做 *shrinkwrap*。

- 將下列的程式碼寫進你的 `index.js` 檔案中：

```js
const color = require('color')

const redHexa = color({ r: 255, g: 0, b: 0 }).hex()

console.log(redHexa)
```

🏁 執行 `yarn start`。 你的終端機應該會印出 `#FF0000`。

恭喜你！你剛剛安裝並且使用了一個套件！

`color` 只是在這一節示範用的簡單套件。我們以後不會再需要它了，所以你可以移除它：

- 執行 `yarn remove color`

## 兩種套件相依

總共有兩種套件相依： `"dependencies"` 和 `"devDependencies"`。

**Dependencies** 是你在應用程式或函數當中會需要用到的相依函式庫。透過 `yarn add [套件名稱]` 安裝。

**Dev Dependencies** 是你在開發的時候會用到的函式庫，例如： Webpack、SASS、linters、測試框架等等。透過 `yarn add --dev [套件名稱]` 安裝。

下一章: [02 - Babel, ES6, ESLint, Flow, Jest, Husky](02-babel-es6-eslint-flow-jest-husky.md#readme)

回到 [目錄](https://github.com/verekia/js-stack-from-scratch#目錄).
