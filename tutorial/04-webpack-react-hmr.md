# 04 - Webpack, React, and Hot Module Replacement

# 04 - Webpack, React, 以及 Hot Module Replacement

Code for this chapter available [here](https://github.com/verekia/js-stack-walkthrough/tree/master/04-webpack-react-hmr).

本章節的程式碼 [點此](https://github.com/verekia/js-stack-walkthrough/tree/master/04-webpack-react-hmr)。

## Webpack

> 💡 **[Webpack](https://webpack.js.org/)** is a *module bundler*. It takes a whole bunch of various source files, processes them, and assembles them into one (usually) JavaScript file called a bundle, which is the only file your client will execute.

> 💡 **[Webpack](https://webpack.js.org/)** 是一個模組打包工具（module bundler）。 他可以將多支不同的來源檔案，處理並且組合成一支可以在終端環境（通常是瀏覽器）執行的 JavaScript 檔案（常見情況下是打包成單一檔案），這裡我們稱它為 bundle。

Let's create some very basic *hello world* and bundle it with Webpack.

讓我們用 Webpack 打包並建立一個基本的"hello world"功能。

- In `src/shared/config.js`, add the following constants:

- 在檔案 `src/shared/config.js` 中加入下列常數：

```js
export const WDS_PORT = 7000

export const APP_CONTAINER_CLASS = 'js-app'
export const APP_CONTAINER_SELECTOR = `.${APP_CONTAINER_CLASS}`
```

- Create an `src/client/index.js` file containing:

- 建立檔案 `src/client/index.js` 並填入下列程式碼：

```js
import 'babel-polyfill'

import { APP_CONTAINER_SELECTOR } from '../shared/config'

document.querySelector(APP_CONTAINER_SELECTOR).innerHTML = '<h1>Hello Webpack!</h1>'
```

If you want to use some of the most recent ES features in your client code, like `Promise`s, you need to include the [Babel Polyfill](https://babeljs.io/docs/usage/polyfill/) before anything else in in your bundle.

如果你想要在你的終端執行環境（通常是瀏覽器）使用一些像是 `Promise` 的最新 ES 功能，你必須在你的 bundle 最開頭引入 [Babel Polyfill](https://babeljs.io/docs/usage/polyfill/)。

- Run `yarn add babel-polyfill`

- 在終端機執行 `yarn add babel-polyfill`

If you run ESLint on this file, it will complain about `document` being undefined.

如果 ESLint 有檢查這支檔案，他會針對 `document` 變數回報出「變數未定義」的警告或錯誤

- Add the following to `env` in your `.eslintrc.json` to allow the use of `window` and `document`:

- 在你的 `.eslintrc.json` 中的 `env` 做下面設定，如此就可以直接寫 `window` 和 `document` 不會報錯

```json
"env": {
  "browser": true,
  "jest": true
}
```

Alright, we now need to bundle this ES6 client app into an ES5 bundle.

接下來我們必須要將 ES6 程式碼打包處理成 ES5 程式碼

- Create a `webpack.config.babel.js` file containing:

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

This file is used to describe how our bundle should be assembled: `entry` is the starting point of our app, `output.filename` is the name of the bundle to generate, `output.path` and `output.publicPath` describe the destination folder and URL. We put the bundle in a `dist` folder, which will contain things that are generated automatically (unlike the declarative CSS we created earlier which lives in `public`). `module.rules` is where you tell Webpack to apply some treatment to some type of files. Here we say that we want all `.js` and `.jsx` (for React) files except the ones in `node_modules` to go through something called `babel-loader`. We also want these two extensions to `resolve`. Finally, we declare a port for Webpack Dev Server.

這支檔案描述了 Webpack 該如何打包我們的檔案。 `entry` 是我們程式的起始點，`output.filename` 是生成 bundle 的名字， `output.path` 跟 `output.publicPath` 分別指明了輸出的「資料夾」跟「完整路徑」，我們將生成的 bundle 放在 `dist` 資料夾，不像在上一章範例中手動直接建立的 CSS 檔案是放在 `public` 資料夾，`dist` 資料夾放置了程式自動生成的檔案。`module.rules` 告訴 Webpack 針對哪些檔案類型要用什麼方式處理，這裡我們告訴 Webpack 用 `babel-loader` 處理除了 `node_modules` 資料夾以外的所有 `.js` 跟 React 的 `.jsx` 檔案。我們也針對兩種附檔名做 `resolve`。最後，指定 Webpack Dev Server 的埠號(port)。

**Note**: The `.babel.js` extension is a Webpack feature to apply our Babel transformations to this config file.

**注意**：有了 `.babel.js` 附檔名可以讓 Babel 轉換功能也應用到這支設定檔。

`babel-loader` is a plugin for Webpack that transpiles your code just like we've been doing since the beginning of this tutorial. The only difference is that this time, the code will end up running in the browser instead of your server.

`babel-loader` 是一個 Webpack 外掛，它讓 Webpack 能像前面第二章教學中那樣轉換你的程式碼，只不過這次轉換出來的程式碼是要運行在瀏覽器而不是你的伺服器中的。

- Run `yarn add --dev webpack webpack-dev-server babel-core babel-loader`

- 在終端機執行 `yarn add --dev webpack webpack-dev-server babel-core babel-loader`

`babel-core` is a peer-dependency of `babel-loader`, so we installed it as well.

`babel-core` 是 `babel-loader` 的 peer-dependency，所以我們必須要一併安裝它。

- Add `/dist/` to your `.gitignore`

- 在 `.gitignore` 增加 `/dist/`

### Tasks update

### 更新任務

In development mode, we are going to use `webpack-dev-server` to take advantage of Hot Module Reloading (later in this chapter), and in production we'll simply use `webpack` to generate bundles. In both cases, the `--progress` flag is useful to display additional information when Webpack is compiling your files. In production, we'll also pass the `-p` flag to `webpack` to minify our code, and the `NODE_ENV` variable set to `production`.

在開發模式下，我們將會使用 `webpack-dev-server` 的 Hot Module Reloading 功能（此功能在本章稍後會解說）。而在生產模式下，我們只使用 `webpack` 指令來生成 bundles。在這兩種模式中我們加入一個實用的 `--progress` 旗標，他能夠讓 Webpack 在編譯過程中呈現額外資訊。生產模式另外增加 `-p` 旗標，告訴 Webpack 最小化我們的程式碼，並且設定 `NODE_ENV` 變數的值為 `production`。

Let's update our `scripts` to implement all this, and improve some other tasks as well:

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

In `dev:start` we explicitly declare file extensions to monitor, `.js` and `.jsx`, and add `dist` in the ignored directories.

在 `dev:start` 任務中，我們明確地宣告監控 `.js` 跟 `.jsx` 兩種附檔名的檔案，並且忽略 `dist` 資料夾。

We created a separate `lint` task and added `webpack.config.babel.js` to the files to lint.

我們建立了一個獨立的 `lint` 任務，並且把檔案 `webpack.config.babel.js` 交給 ESlint 檢查。

- Next, let's create the container for our app in `src/server/render-app.js`, and include the bundle that will be generated:

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

Depending on the environment we're in, we'll include either the Webpack Dev Server bundle, or the production bundle. Note that the path to Webpack Dev Server's bundle is *virtual*, `dist/js/bundle.js` is not actually read from your hard drive in development mode. It's also necessary to give Webpack Dev Server a different port than your main web port.

依據不同的環境，我們將會引入不同的 bundle，開發模式下會引入 Webpack Dev Server bundle；生產模式會引入 production bundle。值得注意的是，Webpack Dev Server bundle 的路徑：`dist/js/bundle.js` 是虛擬的，它並不是真的從你的硬碟中讀取檔案。另外，必須替 Webpack Dev Server 設定一個不同於主要服務的埠號(port)。

- Finally, in `src/server/index.js`, tweak your `console.log` message like so:

- 最後，在檔案 `src/server/index.js` 中稍微改動你的 `console.log` 訊息，如下：

```js
console.log(`Server running on port ${WEB_PORT} ${isProd ? '(production)' :
  '(development).\nKeep "yarn dev:wds" running in an other terminal'}.`)
```

That will give other developers a hint about what to do if they try to just run `yarn start` without Webpack Dev Server.

這樣可以在其他開發者執行了 `yarn start` 卻忘記啟動 Webpack Dev Server 時提醒他。

Alright that was a lot of changes, let's see if everything works as expected:

我們已經做了不少修改變動，來看看是不是都如預期般地運作：

🏁 Run `yarn start` in a terminal. Open an other terminal tab or window, and run `yarn dev:wds` in it. Once Webpack Dev Server is done generating the bundle and its sourcemaps (which should both be ~600kB files) and both processes hang in your terminals, open `http://localhost:8000/` and you should see "Hello Webpack!". Open your Chrome console, and under the Source tab, check which files are included. You should only see `static/css/style.css` under `localhost:8000/`, and have all your ES6 source files under `webpack://./src`. That means sourcemaps are working. In your editor, in `src/client/index.js`, try changing `Hello Webpack!` into any other string. As you save the file, Webpack Dev Server in your terminal should generate a new bundle and the Chrome tab should reload automatically.

🏁 在終端機執行 `yarn start`，然後再開啟終端機的另一個分頁或視窗，並執行 `yarn dev:wds`。一旦兩個終端機畫面都停住並且畫面中顯示 Webpack Dev Server 成功生成 bundle 以及 bundle 的 sourcemaps 時（大約 600KB 大小），打開網址 `http://localhost:8000/`，你應該會看到 "Hello Webpack!"。接著打開 Chrome 的開發者工具，在 Sources 的分頁下檢視哪些檔案被引入，你應該只會看到 `static/css/style.css` 是來自 `localhost:8000/`，其他 ES6 的來源檔案會來自 `webpack://./src`，這代表著 sourcemaps 有成功運作。回到你的編輯器，在檔案 `src/client/index.js` 中把 `Hello Webpack!` 改成任意其他的字，當你儲存時，終端機中的 Webpack Dev Server 應該會生成新的 bundle，並且你的 Chrome 分頁會自動重新整理。

- Kill the previous processes in your terminals with Ctrl+C, then run `yarn prod:build`, and then `yarn prod:start`. Open `http://localhost:8000/` and you should still see "Hello Webpack!". In the Source tab of the Chrome console, you should this time find `static/js/bundle.js` under `localhost:8000/`, but no `webpack://` sources. Click on `bundle.js` to make sure it is minified. Run `yarn prod:stop`.

- 回到終端機，輸入 Ctrl+C 把先前指令執行的行程關掉。接著再執行 `yarn prod:build`，以及 `yarn prod:start`。打開網址 `http://localhost:8000/` ，你會看到 "Hello Webpack!"。在 Chrome 開發者工具中的 Sources 分頁，你應該會看到來源來自 `localhost:8000/` 的檔案 `static/js/bundle.js`，並且已經沒有來自 `webpack://` 的檔案。點選 `bundle.js` 確認程式碼有被最小化。最後，執行 `yarn prod:stop`。

Good job, I know this was quite dense. You deserve a break! The next section is easier.

做得好！我知道這些可能有點複雜，你可以稍微喘口氣，下一個小節會比較簡單。

**Note**: I would recommend to have at least 3 terminals open, one for your Express server, one for the Webpack Dev Server, and one for Git, tests, and general commands like installing packages with `yarn`. Ideally, you should split your terminal screen in multiple panes to see them all.

**注意**：我建議至少同時打開 3 個終端機畫面。一個給 Express server，一個給 Webpack Dev Server，一個給 Git、測試任務或是其他像安裝套件的指令。理想上，你應該將你的終端機視窗分割成多個 pane，如此方便一覽全局。

## React

> 💡 **[React](https://facebook.github.io/react/)** is a library for building user interfaces by Facebook. It uses the **[JSX](https://facebook.github.io/react/docs/jsx-in-depth.html)** syntax to represent HTML elements and components while leveraging the power of JavaScript.

> 💡 **[React](https://facebook.github.io/react/)** 是 Facebook 創造用來打造使用者介面(user interface)的一個函示庫。它使用 **[JSX](https://facebook.github.io/react/docs/jsx-in-depth.html)** 語法來表示 HTML 元素並且借力於 JavaScript 的強大功能。

In this section we are going to render some text using React and JSX.

在這個小節，我們將會用 React 和 JSX 呈現出一些文字。

First, let's install React and ReactDOM:

首先，讓我們安裝 React 和 ReactDOM：

- Run `yarn add react react-dom`

- 在終端機執行 `yarn add react react-dom`

Rename your `src/client/index.js` file into `src/client/index.jsx` and write some React code in it:

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

- Create a `src/client/app.jsx` file containing:

- 建立檔案 `src/client/app.jsx` 並填入：

```js
// @flow

import React from 'react'

const App = () => <h1>Hello React!</h1>

export default App
```

Since we use the JSX syntax here, we have to tell Babel that it needs to transform it as well.

因為使用 JSX 語法，我們必須讓 Babel 也可以轉換它。

- Run `yarn add --dev babel-preset-react` and add `react` to your `.babelrc` file like so:

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

🏁 Run `yarn start` and `yarn dev:wds` and hit `http://localhost:8000`. You should see "Hello React!".

🏁 執行 `yarn start` 和 `yarn dev:wds`，然後打開網址 `http://localhost:8000`。你應該會看到 "Hello React!"。

Now try changing the text in `src/client/app.jsx` to something else. Webpack Dev Server should reload the page automatically, which is pretty neat, but we are going to make it even better.

現在試著改變檔案 `src/client/app.jsx` 中的文字，Webpack Dev Server 應該會優雅地自動重整網頁，不過接下來我們還可以讓它表現得更棒。

## Hot Module Replacement

> 💡 **[Hot Module Replacement](https://webpack.js.org/concepts/hot-module-replacement/)** (*HMR*) is a powerful Webpack feature to replace a module on the fly without reloading the entire page.

> 💡 **[Hot Module Replacement](https://webpack.js.org/concepts/hot-module-replacement/)** (*HMR*) 是一個強大的 Webpack 功能，它能夠做到在不重整網頁的情況下自動替換模組(module)。

To make HMR work with React, we are going to need to tweak a few things.

要搭配 HMR 跟 React，我們將需要做一些調整。

- Run `yarn add react-hot-loader@next`

- 執行 `yarn add react-hot-loader@next`

- Edit your `webpack.config.babel.js` like so:

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

- Edit your `src/client/index.jsx` file:

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

We need to make our `App` a child of `react-hot-loader`'s `AppContainer`, and we need to `require` the next version of our `App` when hot-reloading. To make this  process clean and DRY, we create a little `wrapApp` function that we use in both places it needs to render `App`. Feel free to move the `eslint-disable global-require` to the top of the file to make this more readable.

我們必須讓 `App` 成為 `react-hot-loader` 的 `AppContainer` 的子組件，然後在 hot-reloading 時 `require` 另一版的 `App`。為了讓整個運作過程乾淨並且做到 DRY，這裡建立了一個函數叫 `wrapApp`，他被用在兩個需要 render `App` 的地方。你可以把 `eslint-disable global-require` 那段移到檔案的最開頭讓那個區塊更易於閱讀。

🏁 Restart your `yarn dev:wds` process if it was still running. Open `localhost:8000`. In the Console tab, you should see some logs about HMR. Go ahead and change something in `src/client/app.jsx` and your changes should be reflected in your browser after a few seconds, without any full-page reload!

🏁 如果你還在執行 `yarn dev:wds` 這個行程，重新執行它。打開 `localhost:8000`，在瀏覽器開發者工具中的 Console 分頁你應該會看到一些關於 HMR 的 logs。回去修改檔案 `src/client/app.jsx`，幾秒鐘後，不用經過全頁重整的過程你的改動應該就會反映到瀏覽器上。

Next section: [05 - Redux, Immutable, Fetch](05-redux-immutable-fetch.md#readme)

下一章節：[05 - Redux, Immutable, Fetch](05-redux-immutable-fetch.md#readme)

Back to the [previous section](03-express-nodemon-pm2.md#readme) or the [table of contents](https://github.com/verekia/js-stack-from-scratch#table-of-contents).

回到 [上一章](03-express-nodemon-pm2.md#readme)，或是回 [目錄](../README.md#內容目錄)。
