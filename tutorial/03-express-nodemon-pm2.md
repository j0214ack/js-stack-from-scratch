# 03 - Express, Nodemon, and PM2

本章的原始碼可以從[這裡](https://github.com/verekia/js-stack-walkthrough/tree/master/03-express-nodemon-pm2)取得

在這個階段，我們會建立一個伺服器來呈現我們的網頁應用程式。我們也會替這個伺服器設定好開發環境以及正式環境。


## Express

> 💡 **[Express](http://expressjs.com/)** 是目前最受歡迎的 Node 網頁應用程式框架。它提供了極簡化的 API ，而它的功能可以透過 middleware 來擴充。

我們來設置一個極小化的 Express 伺服器來提供 HTML 頁面和一些 CSS 。

- 刪除 `src` 裡面所有的檔案

- 創建一個 `public/css/style.css` 檔案，裡面寫：

```css
body {
  width: 960px;
  margin: auto;
  font-family: sans-serif;
}

h1 {
  color: limegreen;
}
```

- 創建一個空的 `src/client/` 資料夾

- 創建一個空的 `src/shared/` 資料夾

這個資料夾是我們放 *同構(isomorphic) / 通用* JavaScript 程式碼的地方。也就是在客戶端與伺服器端同時都會用到的程式碼。像 *routes* 就是非常適合用來共享的程式碼範例。在稍後的教學中裡，我們會製作一個非同步的呼叫，到時候就會提到。現在為了示範，我們先放一些設置用的常數即可。


- 創建一個 `src/shared/config.js` 檔案，內含：


```js
// @flow

export const WEB_PORT = process.env.PORT || 8000
export const STATIC_PATH = '/static'
export const APP_NAME = 'Hello App'
```

如果用來運行你的程式的 Node 程序有設置 `process.env.PORT` 環境變數的話（例如當你用 Heroku 部署的時候）， 這段程式碼就會用那個環境變數作為連接埠埠號，如果沒有該環境變數，我們預設埠號為  `8000`。

- 創建一個 `src/shared/util.js` 檔案，內含：

```js
// @flow

// eslint-disable-next-line import/prefer-default-export
export const isProd = process.env.NODE_ENV === 'production'
```

這是一個簡單的工具程式，用得知我們現在是不是在正式環境。 `// eslint-disable-next-line import/prefer-default-export` 註解是因為我們只有一個 `named export` ，而 `eslint` 的 `prefer-default-export` 規則會在這種狀況下建議我們用 `export default` 。當我們增加其他 `named export` 到這個檔案之後，就可以移除這行註解了。

- 執行 `yarn add express compression`

`compression` 是一個 Express 的 middleware ，用來在伺服器上啟用 Gzip 壓縮。

- 創建一個 `src/server/index.js` 檔案，內含：

```js
// @flow

import compression from 'compression'
import express from 'express'

import { APP_NAME, STATIC_PATH, WEB_PORT } from '../shared/config'
import { isProd } from '../shared/util'
import renderApp from './render-app'

const app = express()

app.use(compression())
app.use(STATIC_PATH, express.static('dist'))
app.use(STATIC_PATH, express.static('public'))

app.get('/', (req, res) => {
  res.send(renderApp(APP_NAME))
})

app.listen(WEB_PORT, () => {
  // eslint-disable-next-line no-console
  console.log(`Server running on port ${WEB_PORT} ${isProd ? '(production)' : '(development)'}.`)
})
```

這段沒什麼特別的，主要就是 Express 版的 Hello World 教學，再加上一些匯入。我們用了兩個不同的靜態檔案資料夾：`dist` 和 `public` ，前者放自動生成的檔案，後者是我們主動放置的檔案。

- 創建一個 `src/server/render-app.js` 檔案，內含：

```js
// @flow

import { STATIC_PATH } from '../shared/config'

const renderApp = (title: string) =>
`<!doctype html>
<html>
  <head>
    <title>${title}</title>
    <link rel="stylesheet" href="${STATIC_PATH}/css/style.css">
  </head>
  <body>
    <h1>${title}</h1>
  </body>
</html>
`

export default renderApp
```

你知道通常後端都會有 *樣版引擎* 對吧？現在這幾乎要過時了，因為 JavaScript 現在支援樣板字串。這裡我們創造了一個函數，將 `title` 當作參數，並且將它注入到這個頁面的樣板字串之中的 `title` 和 `h1` 標籤，然後加注入後的完整 HTML 字串回傳。同時我們用了 `STATIC_PATH` 常數，作為所有靜態資產的基礎路徑。

### Atom 中的 HTML 樣板字串語法高亮 (選擇性)
在你的編輯器中，你是有可能將在樣板字串中的 HTML 語法高亮的。以 Atom 為例，你可以在字串之前前綴一個 `html` 標註（或任何以 `html` 結尾的標註，像是 `ilovehtml` ）， Atom 就會自動將該字串的內容以 HTML 語法高亮。我有時會同時引入 `common-tags` 函式庫裡的 `html` 標註來一石二鳥。（譯注：原作者在這裡沒有說清楚， [common-tags](https://github.com/declandewet/common-tags) 函式庫裡面內含許多樣板 `tag` ，樣板字串之前具有 `tag` 屬性是 ES 2015 的功能，可以決定樣板字串輸出的格式，而 `common-tags` 裡的各種 `tag` 是讓輸出格式好看用的自訂模板。）

```js
import { html } from `common-tags`

const template = html`
<div>Wow, colors!</div>
`
```

我沒有在這個教學的樣板文件放上這個小技巧，因為目前看來只有 Atom 可以用，而且不甚理想。或許一些使用 Atom 的讀者會覺得有用吧。

無論如何，回到正題。


- 在 `package.json` 裡，將你的 `start` 腳本換成： `"start": "babel-node src/server",`


🏁 執行 `yarn start` ，然後在你的瀏覽器裡拜訪 `localhost:8000` 。如果一切順利，你應該會看到一個空白頁面，有著綠色的 "Hello App" 標題，還有 "Hello App" 分頁名稱。

**備註**：有一些程序 - 通常是一些等待事情發生的程序，例如伺服器 - 會讓你無法在終端機中輸入指令，直到他們終止。要中斷這些程序並且將讓你可以繼續下指令，按下 **Ctrl+C** 。如果你想要在輸入指令的同時讓伺服器繼續運作，你也可以開一個新的終端機分頁。當然，你也可以讓這些程序在背景運作，不過這已經超出了本教學的介紹範圍了。


## Nodemon

> 💡 **[Nodemon](https://nodemon.io/)** 是一個讓你的 Node 伺服器在有檔案更新的時候，自動重新啟動的功能性模組。

當我們在 **開發** 環境的時候，我們將會使用 Nodemon 。

- 執行 `yarn add --dev nodemon`

- 在 `package.json` 裡把你的 `scripts` 改成:

```json
"start": "yarn dev:start",
"dev:start": "nodemon --ignore lib --exec babel-node src/server",
```

`start` 現在指向另外一項任務： `dev:start`。這又給我們添加了另外一個抽象層，讓我們可以調整預設的任務。


在 `dev:start` 裡的 `--ignore lib` 旗標，是當 `lib` 資料夾產生變動的時候，讓伺服器 *不要* 重新啟動的指令。你現在還沒有這個資料夾，我們會在本章的下一個段落產生它，到時候這就會看起來比較有道理一點了。 Nodemon 通常會執行 `node` 可執行檔。但既然我們用的是 Babel ，我們會跟 Nodemon 說請使用 `babel-node`。如此才能看得懂 ES6/Flow 程式碼。


🏁 執行 `yarn start` 而且打開 `localhost:8000`。試著改變 `src/shared/config.js` 裡的 `APP_NAME` 常數，這應該會觸發伺服器重新啟動，你可以在終端機裡看到。重新整理頁面以看見更新後的標題。請注意，伺服器自動重新啟動跟 *熱模組切換* (HMR, hot module replacement) 是不同的，後者是當頁面中的組件變更的時候會即時在頁面上更新。現在我們仍需要手動重整頁面，不過至少我們不需要關掉程序在手動重新開起來看到更動了。熱模組切換會在下一章節裡介紹。

## PM2

> 💡 **[PM2](http://pm2.keymetrics.io/)** 是 Node 的程序管理員。它讓我們在正式環境裡讓我們的程序持續運作，同時給我們許多管理及監控程序的功能。

當我們進入正式環境後，我們將會使用 PM2。

- 執行 `yarn add --dev pm2`

在正式環境裡，我們希望伺服器的效能越高越好。 `babel-node` 會在每次執行的時候，觸發整個 Babel 的檔案編譯過程，這可不是你想要在正式環境裡發生的事。我們需要 Babel 在事前替我們完成這些事情，讓我們的伺服器可以直接送出編譯好的舊 ES5 檔案。

Babel 的一個主要功能是將 ES6 程式碼的資料夾(通常是 `src`)轉譯成另一個都是 ES5 程式碼的資料夾(通常是 `lib`)

`lib` 資料夾是自動產生的。一個好習慣是在每次編譯新的程式碼前都將它清理乾淨，應該它可能內含不該出現的舊檔案。`rimraf` 是一個簡單乾淨的跨平台套件，用來刪除這些檔案。

- 執行 `yarn add --dev rimraf`

我們來把 `prod:build` 任務加到我們的 `package.json` 吧：

```json
"prod:build": "rimraf lib && babel src -d lib --ignore .test.js",
```

- 執行 `yarn prod:build` 之後應該會產生一個 `lib` 資料夾，裡面有轉譯過後的程式碼，除了以 `.test.js` 結尾的檔案 (`.test.jsx` 同時也會被忽略)

- 將 `/lib/` 將到 `.gitignore` 中

最後一件事：我們將會把 `NODE_ENV` 環境變數傳入 PM2 程序中。在 Unix 系統下，我們會執行 `NODE_ENV=production pm2`，但在 Windows 當中的語法又有點不同。我們會用一個叫做 `cross-env` 的小套件來讓這個語法也能在 Windows 底下工作。

- 執行 `yarn add --dev cross-env`

更新 `package.json` ：

```json
"scripts": {
  "start": "yarn dev:start",
  "dev:start": "nodemon --ignore lib --exec babel-node src/server",
  "prod:build": "rimraf lib && babel src -d lib --ignore .test.js",
  "prod:start": "cross-env NODE_ENV=production pm2 start lib/server && pm2 logs",
  "prod:stop": "pm2 delete all",
  "test": "eslint src && flow && jest --coverage",
  "precommit": "yarn test",
  "prepush": "yarn test"
},
```

🏁 執行 `yarn prod:build`，然後 `yarn prod:start`。 PM2 應該會顯示一個運作中的程序。在瀏覽器中前往 `http://localhost:8000` ，你應該會看到你的程式。你的終端機應該會顯示紀錄文字 "Server running on port 8000 (production)."。 當你用 PM2 的時候，你的程序是在背景運作的。如果你按下 Ctrl+C，這會終止 `pm2 logs` 指令，也就是我們在 `prod:start` 任務鍊中的最後一個指令，但伺服器仍會繼續運作並顯示頁面。如果你希望終止伺服器，請執行 `yarn prod:stop` 。

現在我們有了 `prod:build` 任務，如果我們在每次推送程式碼到資源庫之前，都能檢查是否能 build 成功的話就太棒了。我建議在 `prepush` 任務裡加上它：

```json
"prepush": "yarn test && yarn prod:build"
```

🏁 執行 `yarn prepush`，或直接推送你的檔案來觸發這個任務。

**備註**：我們還沒有任何測試，所以 Jest 會發出一些抱怨，暫時先不用理它。

下一章： [04 - Webpack, React, HMR](04-webpack-react-hmr.md#readme)

回到 [前一章](02-babel-es6-eslint-flow-jest-husky.md#readme) 或回到 [內容目錄](https://github.com/verekia/js-stack-from-scratch#內容目錄).
