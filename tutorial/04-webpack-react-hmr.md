# 04 - Webpack, React, 以及 Hot Module Replacement

本章的程式碼 [點此](https://github.com/verekia/js-stack-walkthrough/tree/master/04-webpack-react-hmr)。

## Webpack

> 💡 **[Webpack](https://webpack.js.org/)** 是一個模組打包工具（module bundler）。 他可以將多支不同的來源檔案，處理並且組合成一支可以在終端環境（通常是瀏覽器）執行的 JavaScript 檔案（常見情況下是打包成單一檔案），這裡我們稱它為 bundle。

讓我們用 Webpack 打包並建立一個基本的"hello world"功能。

- 在檔案 `src/shared/config.js` 中加入下列常數：

```js
export const WDS_PORT = 7000

export const APP_CONTAINER_CLASS = 'js-app'
export const APP_CONTAINER_SELECTOR = `.${APP_CONTAINER_CLASS}`
```

- 建立檔案 `src/client/index.js` 並填入下列程式碼：

```js
import 'babel-polyfill'

import { APP_CONTAINER_SELECTOR } from '../shared/config'

document.querySelector(APP_CONTAINER_SELECTOR).innerHTML = '<h1>Hello Webpack!</h1>'
```

如果你想要在你的終端執行環境（通常是瀏覽器）使用一些像是 `Promise` 的最新 ES 功能，你必須在你的 bundle 最開頭引入 [Babel Polyfill](https://babeljs.io/docs/usage/polyfill/)。

- 在終端機執行 `yarn add babel-polyfill`

如果 ESLint 有檢查這支檔案，他會針對 `document` 變數回報出「變數未定義」的警告或錯誤

- 在你的 `.eslintrc.json` 中的 `env` 做下面設定，如此就可以直接寫 `window` 和 `document` 不會報錯

```json
"env": {
  "browser": true,
  "jest": true
}
```

現在我們將要把 ES6 程式碼打包處理成 ES5 程式碼

- 建立檔案 `webpack.config.babel.js` 並填入下列程式碼：

```js
// @flow

import path from 'path'

import { WDS_PORT } from './src/shared/config'
import { isProd } from './src/shared/util'

export default {
  entry: [
    './src/client',
  ],
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist/js'),
    publicPath: `http://localhost:${WDS_PORT}/dist/js/`,
  },
  module: {
    rules: [
      { test: /\.(js|jsx)$/, use: 'babel-loader', exclude: /node_modules/ },
    ],
  },
  devtool: isProd ? false : 'source-map',
  resolve: {
    extensions: ['.js', '.jsx'],
  },
  devServer: {
    port: WDS_PORT,
  },
}
```

這支檔案描述了 Webpack 該如何打包我們的檔案。 `entry` 是我們程式的起始點，`output.filename` 是生成 bundle 的名字， `output.path` 跟 `output.publicPath` 分別指明了輸出的「資料夾」跟「完整路徑」，我們將生成的 bundle 放在 `dist` 資料夾，不像在上一章範例中手動直接建立的 CSS 檔案是放在 `public` 資料夾，`dist` 資料夾放置了程式自動生成的檔案。`module.rules` 告訴 Webpack 針對哪些檔案類型要用什麼方式處理，這裡我們告訴 Webpack 用 `babel-loader` 處理除了 `node_modules` 資料夾以外的所有 `.js` 跟 React 的 `.jsx` 檔案。我們也針對兩種附檔名做 `resolve`。最後，指定 Webpack Dev Server 的埠號(port)。

**注意**：有了 `.babel.js` 附檔名可以讓 Babel 轉換功能也應用到這支設定檔。

`babel-loader` 是一個 Webpack 外掛，它讓 Webpack 能像前面第二章教學中那樣轉換你的程式碼，只不過這次轉換出來的程式碼是要運行在瀏覽器而不是你的伺服器中的。

- 在終端機執行 `yarn add --dev webpack webpack-dev-server babel-core babel-loader`

`babel-core` 是 `babel-loader` 的 peer-dependency，所以我們必須要一併安裝它。

- 在 `.gitignore` 增加 `/dist/`

### 更新任務

在開發模式下，我們將會使用 `webpack-dev-server` 的 Hot Module Reloading 功能（此功能在本章稍後會解說）。而在生產模式下，我們只使用 `webpack` 指令來生成 bundles。在這兩種模式中我們加入一個實用的 `--progress` 旗標，他能夠讓 Webpack 在編譯過程中呈現額外資訊。生產模式另外增加 `-p` 旗標，告訴 Webpack 最小化我們的程式碼，並且設定 `NODE_ENV` 變數的值為 `production`。

讓我們將設定更新到 `scripts` 中，並且也調整其他任務：

```json
"scripts": {
  "start": "yarn dev:start",
  "dev:start": "nodemon -e js,jsx --ignore lib --ignore dist --exec babel-node src/server",
  "dev:wds": "webpack-dev-server --progress",
  "prod:build": "rimraf lib dist && babel src -d lib --ignore .test.js && cross-env NODE_ENV=production webpack -p --progress",
  "prod:start": "cross-env NODE_ENV=production pm2 start lib/server && pm2 logs",
  "prod:stop": "pm2 delete all",
  "lint": "eslint src webpack.config.babel.js --ext .js,.jsx",
  "test": "yarn lint && flow && jest --coverage",
  "precommit": "yarn test",
  "prepush": "yarn test && yarn prod:build"
},
```

在 `dev:start` 任務中，我們明確地宣告監控 `.js` 跟 `.jsx` 兩種附檔名的檔案，並且忽略 `dist` 資料夾。

我們建立了一個獨立的 `lint` 任務，並且把檔案 `webpack.config.babel.js` 交給 ESlint 檢查。

- 接著，讓我們建立檔案 `src/server/render-app.js` 用來當作我們應用程式的容器，並在其中引入將會被打包好的 bundle：

```js
// @flow

import { APP_CONTAINER_CLASS, STATIC_PATH, WDS_PORT } from '../shared/config'
import { isProd } from '../shared/util'

const renderApp = (title: string) =>
`<!doctype html>
<html>
  <head>
    <title>${title}</title>
    <link rel="stylesheet" href="${STATIC_PATH}/css/style.css">
  </head>
  <body>
    <div class="${APP_CONTAINER_CLASS}"></div>
    <script src="${isProd ? STATIC_PATH : `http://localhost:${WDS_PORT}/dist`}/js/bundle.js"></script>
  </body>
</html>
`

export default renderApp
```

依據不同的環境，我們將會引入不同的 bundle，開發模式下會引入 Webpack Dev Server bundle；生產模式會引入 production bundle。值得注意的是，Webpack Dev Server bundle 的路徑：`dist/js/bundle.js` 是虛擬的，它並不是真的從你的硬碟中讀取檔案。另外，必須替 Webpack Dev Server 設定一個不同於主要服務的埠號(port)。

- 最後，在檔案 `src/server/index.js` 中稍微改動你的 `console.log` 訊息，如下：

```js
console.log(`Server running on port ${WEB_PORT} ${isProd ? '(production)' :
  '(development).\nKeep "yarn dev:wds" running in an other terminal'}.`)
```

這樣可以在其他開發者執行了 `yarn start` 卻忘記啟動 Webpack Dev Server 時提醒他。

我們已經做了不少修改變動，來看看是不是都如預期般地運作：

🏁 在終端機執行 `yarn start`，然後再開啟終端機的另一個分頁或視窗，並執行 `yarn dev:wds`。一旦兩個終端機畫面都停住並且畫面中顯示 Webpack Dev Server 成功生成 bundle 以及 bundle 的 sourcemaps 時（大約 600KB 大小），打開網址 `http://localhost:8000/`，你應該會看到 "Hello Webpack!"。接著打開 Chrome 的開發者工具，在 Sources 的分頁下檢視哪些檔案被引入，你應該只會看到 `static/css/style.css` 是來自 `localhost:8000/`，其他 ES6 的來源檔案會來自 `webpack://./src`，這代表著 sourcemaps 有成功運作。回到你的編輯器，在檔案 `src/client/index.js` 中把 `Hello Webpack!` 改成任意其他的字，當你儲存時，終端機中的 Webpack Dev Server 應該會生成新的 bundle，並且你的 Chrome 分頁會自動重新整理。

- 回到終端機，輸入 Ctrl+C 把先前指令執行的行程關掉。接著再執行 `yarn prod:build`，以及 `yarn prod:start`。打開網址 `http://localhost:8000/` ，你會看到 "Hello Webpack!"。在 Chrome 開發者工具中的 Sources 分頁，你應該會看到來源來自 `localhost:8000/` 的檔案 `static/js/bundle.js`，並且已經沒有來自 `webpack://` 的檔案。點選 `bundle.js` 確認程式碼有被最小化。最後，執行 `yarn prod:stop`。

做得好！我知道這些可能有點複雜，你可以稍微喘口氣，下一個章節會比較簡單。

**注意**：我建議至少同時打開 3 個終端機畫面。一個給 Express server，一個給 Webpack Dev Server，一個給 Git、測試任務或是其他像安裝套件的指令。理想上，你應該將你的終端機視窗分割成多個 pane，如此方便一覽全局。

## React

> 💡 **[React](https://facebook.github.io/react/)** 是 Facebook 創造用來打造使用者介面(user interface)的一個函示庫。它使用 **[JSX](https://facebook.github.io/react/docs/jsx-in-depth.html)** 語法來表示 HTML 元素並且借力於 JavaScript 的強大功能。

在這個章節，我們將會用 React 和 JSX 呈現出一些文字。

首先，讓我們安裝 React 和 ReactDOM：

- 在終端機執行 `yarn add react react-dom`

將檔案 `src/client/index.js` 改名成 `src/client/index.jsx` 並且寫入一些 React 的程式碼：

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'

import App from './app'
import { APP_CONTAINER_SELECTOR } from '../shared/config'

ReactDOM.render(<App />, document.querySelector(APP_CONTAINER_SELECTOR))
```

- 建立檔案 `src/client/app.jsx` 並填入：

```js
// @flow

import React from 'react'

const App = () => <h1>Hello React!</h1>

export default App
```

因為使用 JSX 語法，我們必須讓 Babel 也可以轉換它。

- 執行 `yarn add --dev babel-preset-react`，並且把 `react` 設定到 `.babelrc` 中，如下：

```json
{
  "presets": [
    "env",
    "flow",
    "react"
  ]
}
```

🏁 執行 `yarn start` 和 `yarn dev:wds`，然後打開網址 `http://localhost:8000`。你應該會看到 "Hello React!"。

現在試著改變檔案 `src/client/app.jsx` 中的文字，Webpack Dev Server 應該會優雅地自動重整網頁，不過接下來我們還可以讓它表現得更棒。

## Hot Module Replacement

> 💡 **[Hot Module Replacement](https://webpack.js.org/concepts/hot-module-replacement/)** (*HMR*) 是一個強大的 Webpack 功能，它能夠做到在不重整網頁的情況下自動替換模組(module)。

要搭配 HMR 跟 React，我們將需要做一些調整。

- 執行 `yarn add react-hot-loader@next`

- 編輯你的 `webpack.config.babel.js`，如下：

```js
import webpack from 'webpack'
// [...]
entry: [
  'react-hot-loader/patch',
  './src/client',
],
// [...]
devServer: {
  port: WDS_PORT,
  hot: true,
},
plugins: [
  new webpack.optimize.OccurrenceOrderPlugin(),
  new webpack.HotModuleReplacementPlugin(),
  new webpack.NamedModulesPlugin(),
  new webpack.NoEmitOnErrorsPlugin(),
],
```

- 編輯你的 `src/client/index.jsx` 檔案：

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'
import { AppContainer } from 'react-hot-loader'

import App from './app'
import { APP_CONTAINER_SELECTOR } from '../shared/config'

const rootEl = document.querySelector(APP_CONTAINER_SELECTOR)

const wrapApp = AppComponent =>
  <AppContainer>
    <AppComponent />
  </AppContainer>

ReactDOM.render(wrapApp(App), rootEl)

if (module.hot) {
  // flow-disable-next-line
  module.hot.accept('./app', () => {
    // eslint-disable-next-line global-require
    const NextApp = require('./app').default
    ReactDOM.render(wrapApp(NextApp), rootEl)
  })
}
```

我們必須讓 `App` 成為 `react-hot-loader` 的 `AppContainer` 的子組件，然後在 hot-reloading 時 `require` 另一版的 `App`。為了讓整個運作過程乾淨並且做到 DRY，這裡建立了一個函數叫 `wrapApp`，他被用在兩個需要 render `App` 的地方。你可以把 `eslint-disable global-require` 那段移到檔案的最開頭讓那個區塊更易於閱讀。

🏁 如果你還在執行 `yarn dev:wds` 這個行程，重新執行它。打開 `localhost:8000`，在瀏覽器開發者工具中的 Console 分頁你應該會看到一些關於 HMR 的 logs。回去修改檔案 `src/client/app.jsx`，幾秒鐘後，不用經過全頁重整的過程你的改動應該就會反映到瀏覽器上。

下一章：[05 - Redux, Immutable, Fetch](05-redux-immutable-fetch.md#readme)

回到 [上一章](03-express-nodemon-pm2.md#readme)，或是回 [目錄](../README.md#目錄)。
