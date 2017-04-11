# 06 - React Router, Server-Side Rendering, and Helmet

本章節的程式碼[點此](https://github.com/verekia/js-stack-walkthrough/tree/master/06-react-router-ssr-helmet).

我們將在本章中為我們的應用程式建立不同的頁面，並且在頁面之間瀏覽

## React Router

> 💡 **[React Router](https://reacttraining.com/react-router/)** 是讓你的 React 應用程式在不同頁面之間瀏覽的函式庫。他可以同時在用戶端與伺服器端使用。

React Router 目前有第四版的重大更新，但仍然處於 beta 階段。我希望這份教學有最新的資訊，我們將會在這裡使用第四版。

- 執行 `yarn add react-router@next react-router-dom@next`

在用戶端，我們首先需要將應用程式包裹一層 `BrowserRouter` 元件

- 更新你的 `src/client/index.jsx` 如下：

```js
// [...]
import { BrowserRouter } from 'react-router-dom'
// [...]
const wrapApp = (AppComponent, reduxStore) =>
  <Provider store={reduxStore}>
    <BrowserRouter>
      <AppContainer>
        <AppComponent />
      </AppContainer>
    </BrowserRouter>
  </Provider>
```

## 頁面

我們的應用將會有四個頁面：

- 首頁
- Hello 頁面，上面有一個按鈕還有訊息，用來呈現同步的 `action`
- 非同步 Hello 頁面，上面有一個按鈕還有訊息，用來呈現非同步的 `action`
-  404 "Not Found" 頁面

- 建立一個 `src/client/component/page/home.jsx` 檔案，內含：

```js
// @flow

import React from 'react'

const HomePage = () => <p>Home</p>

export default HomePage
```

- 建立一個 `src/client/component/page/hello.jsx` 檔案，內含：

```js
// @flow

import React from 'react'

import HelloButton from '../../container/hello-button'
import Message from '../../container/message'

const HelloPage = () =>
  <div>
    <Message />
    <HelloButton />
  </div>

export default HelloPage

```

- 建立一個 `src/client/component/page/hello-async.jsx` 檔案，內含：

```js
// @flow

import React from 'react'

import HelloAsyncButton from '../../container/hello-async-button'
import MessageAsync from '../../container/message-async'

const HelloAsyncPage = () =>
  <div>
    <MessageAsync />
    <HelloAsyncButton />
  </div>

export default HelloAsyncPage
```

- 建立一個 `src/client/component/page/not-found.jsx` 檔案，內含：

```js
// @flow

import React from 'react'

const NotFoundPage = () => <p>Page not found</p>

export default NotFoundPage
```

## 瀏覽

在共用的設定檔案裡增加路徑

- 編輯你的 `src/shared/routes.js` 如下：

```js
// @flow

export const HOME_PAGE_ROUTE = '/'
export const HELLO_PAGE_ROUTE = '/hello'
export const HELLO_ASYNC_PAGE_ROUTE = '/hello-async'
export const NOT_FOUND_DEMO_PAGE_ROUTE = '/404'

export const helloEndpointRoute = (num: ?number) => `/ajax/hello/${num || ':num'}`
```

路徑 `/404` 會被放在導覽連結中，是為了展示當有人點擊壞了的連結之後或發生什麼事

- 建立一個 `src/client/component/nav.jsx` 檔案，內含：

```js
// @flow

import React from 'react'
import { NavLink } from 'react-router-dom'
import {
  HOME_PAGE_ROUTE,
  HELLO_PAGE_ROUTE,
  HELLO_ASYNC_PAGE_ROUTE,
  NOT_FOUND_DEMO_PAGE_ROUTE,
} from '../../shared/routes'

const Nav = () =>
  <nav>
    <ul>
      {[
        { route: HOME_PAGE_ROUTE, label: 'Home' },
        { route: HELLO_PAGE_ROUTE, label: 'Say Hello' },
        { route: HELLO_ASYNC_PAGE_ROUTE, label: 'Say Hello Asynchronously' },
        { route: NOT_FOUND_DEMO_PAGE_ROUTE, label: '404 Demo' },
      ].map(link => (
        <li key={link.route}>
          <NavLink to={link.route} activeStyle={{ color: 'limegreen' }} exact>{link.label}</NavLink>
        </li>
      ))}
    </ul>
  </nav>

export default Nav
```

在這裡我們只不過是創造了一堆使用我們之前宣告的路徑的 `NavLink`

- 最後，編輯 `src/client/app.jsx` 如下：

```js
// @flow

import React from 'react'
import { Switch } from 'react-router'
import { Route } from 'react-router-dom'
import { APP_NAME } from '../shared/config'
import Nav from './component/nav'
import HomePage from './component/page/home'
import HelloPage from './component/page/hello'
import HelloAsyncPage from './component/page/hello-async'
import NotFoundPage from './component/page/not-found'
import {
  HOME_PAGE_ROUTE,
  HELLO_PAGE_ROUTE,
  HELLO_ASYNC_PAGE_ROUTE,
} from '../shared/routes'

const App = () =>
  <div>
    <h1>{APP_NAME}</h1>
    <Nav />
    <Switch>
      <Route exact path={HOME_PAGE_ROUTE} render={() => <HomePage />} />
      <Route path={HELLO_PAGE_ROUTE} render={() => <HelloPage />} />
      <Route path={HELLO_ASYNC_PAGE_ROUTE} render={() => <HelloAsyncPage />} />
      <Route component={NotFoundPage} />
    </Switch>
  </div>

export default App
```

🏁 執行 `yarn start` 以及 `yarn dev:wds` 然後打開 `http://localhost:8000` 並點擊這些連結來在不同的頁面之間瀏覽。你應該可以看到網址列動態地改變了。用上一頁來切換頁面，應該可以看到瀏覽器歷史如期地運作。

現在，假設我們瀏覽到 `http://localhost:8000/hello` ，接著按下重新整理。你現在會得到 404 ，因為我們的 Express 伺服器只會對 `/` 做出回應。目前我們在頁面切換的時候，只不過是在前端切換而已，讓我們加上伺服器端呈現(server-side rendering)來得到預期的行為。

## 伺服器端呈現(Server-Side Rendering)

> 💡 **Server-Side Rendering(SSR)** 的意思是讓我們的應用程式在一開始就能載入完畢，不必依賴客戶端瀏覽器的 JavaScript 運作來呈現。

SSR 對於搜尋引擎優化(SEO)至關重要，也且也能提供用戶較佳的使用者體驗，因為他們馬上就能看到應用程式的內容

第一件事情是將我們的客戶端程式搬遷到程式庫裡共用(shared / isomorphic / universal) 的地方。因為我們的伺服器也一樣要開始呈現 React 應用了。
(譯註：目前社群傾向稱呼這樣的作法為 isomorphic javascript ，很多時候你會看到資料夾名稱是 `shared`，也有一些人稱呼這種作法為 universal ，所以原作者才在此連續放了三個名詞，怕讀者日後看到其他討論時搞不清楚。)

### `shared` 大遷徙

- 將所有在 `client` 當中的檔案移動到 `shared` ，除了 `src/client/index.js`

我們需要調整成堆的 `import`：

- 在 `src/client/index.jsx` 裡，將三處 `'./app'` 換成 `'../shared/app'`，將 `'./reducer/hello'` 改為 `'../shared/reducer/hello'`

- 在 `src/shared/app.jsx` 裡，將 `'../shared/routes'` 換成 `'./routes'` ，將 `'../shared/config'` 改為 `'./config'`

- 在 `src/shared/component/nav.jsx` 裡， `'../../shared/routes'` 換成 `'../routes'`

### 伺服器端的更動

- 建立一個  `src/server/routing.js` 檔案，內含：

```js
// @flow

import {
  homePage,
  helloPage,
  helloAsyncPage,
  helloEndpoint,
} from './controller'

import {
  HOME_PAGE_ROUTE,
  HELLO_PAGE_ROUTE,
  HELLO_ASYNC_PAGE_ROUTE,
  helloEndpointRoute,
} from '../shared/routes'

import renderApp from './render-app'

export default (app: Object) => {
  app.get(HOME_PAGE_ROUTE, (req, res) => {
    res.send(renderApp(req.url, homePage()))
  })

  app.get(HELLO_PAGE_ROUTE, (req, res) => {
    res.send(renderApp(req.url, helloPage()))
  })

  app.get(HELLO_ASYNC_PAGE_ROUTE, (req, res) => {
    res.send(renderApp(req.url, helloAsyncPage()))
  })

  app.get(helloEndpointRoute(), (req, res) => {
    res.json(helloEndpoint(req.params.num))
  })

  app.get('/500', () => {
    throw Error('Fake Internal Server Error')
  })

  app.get('*', (req, res) => {
    res.status(404).send(renderApp(req.url))
  })

  // eslint-disable-next-line no-unused-vars
  app.use((err, req, res, next) => {
    // eslint-disable-next-line no-console
    console.error(err.stack)
    res.status(500).send('Something went wrong!')
  })
}
```

這個檔案就是我們用來處理請求與回應的地方。對於業務邏輯的呼叫將外派給不同的 `controller` 模組負責。


**備註**： 你會發現非常多 React Router 的範例使用 `*` 作為伺服器端的路徑，讓整個路徑處理交給 React Router。不過當所有的請求都通過同樣的函式的話，會使得 MVC 風格的網站不容易實作。我們不會在這份教學這樣做，我們會明白地宣告路徑以及他們專屬的回應，以便我們從資料庫取得資料必且傳遞給指定的頁面。

- 建立一個 `src/server/controller.js` 檔案，內含：

```js
// @flow

export const homePage = () => null

export const helloPage = () => ({
  hello: { message: 'Server-side preloaded message' },
})

export const helloAsyncPage = () => ({
  hello: { messageAsync: 'Server-side preloaded message for async page' },
})

export const helloEndpoint = (num: number) => ({
  serverMessage: `Hello from the server! (received ${num})`,
})
```

這就是我們的 `controller` 了。通常這是處理業務邏輯以及呼叫資料庫的地方。但目前我們只會將一些結果寫死。這些結果會回傳給 `routing` 模組，用以初始化我們的伺服器端 Redux store 。

- 創建 `src/server/init-store.js` 檔案，內含：

```js
// @flow

import * as Immutable from 'immutable'
import { createStore, combineReducers, applyMiddleware } from 'redux'
import thunkMiddleware from 'redux-thunk'

import helloReducer from '../shared/reducer/hello'

const initStore = (plainPartialState: ?Object) => {
  const preloadedState = plainPartialState ? {} : undefined

  if (plainPartialState && plainPartialState.hello) {
    // flow-disable-next-line
    preloadedState.hello = helloReducer(undefined, {})
      .merge(Immutable.fromJS(plainPartialState.hello))
  }

  return createStore(combineReducers({ hello: helloReducer }),
    preloadedState, applyMiddleware(thunkMiddleware))
}

export default initStore
```

除了呼叫 `createStore` 以及使用 middleware 之外，我們只是將從 `controller` 得到的 JS 物件傳遞到預設的 Redux state 並轉換成 Immutable 物件

- 編輯 `src/server/index.js` 如下：

```js
// @flow

import compression from 'compression'
import express from 'express'

import routing from './routing'
import { WEB_PORT, STATIC_PATH } from '../shared/config'
import { isProd } from '../shared/util'

const app = express()

app.use(compression())
app.use(STATIC_PATH, express.static('dist'))
app.use(STATIC_PATH, express.static('public'))

routing(app)

app.listen(WEB_PORT, () => {
  // eslint-disable-next-line no-console
  console.log(`Server running on port ${WEB_PORT} ${isProd ? '(production)' :
    '(development).\nKeep "yarn dev:wds" running in an other terminal'}.`)
})
```

這裡沒有什麼特別的地方，我們呼叫了 `routing(app)` 而不是直接在這裡實作路徑處理

- 重新將 `src/server/render-app.js` 命名為 `src/server/render-app.jsx`，並編輯如下：

```js
// @flow

import React from 'react'
import ReactDOMServer from 'react-dom/server'
import { Provider } from 'react-redux'
import { StaticRouter } from 'react-router'

import initStore from './init-store'
import App from './../shared/app'
import { APP_CONTAINER_CLASS, STATIC_PATH, WDS_PORT } from '../shared/config'
import { isProd } from '../shared/util'

const renderApp = (location: string, plainPartialState: ?Object, routerContext: ?Object = {}) => {
  const store = initStore(plainPartialState)
  const appHtml = ReactDOMServer.renderToString(
    <Provider store={store}>
      <StaticRouter location={location} context={routerContext}>
        <App />
      </StaticRouter>
    </Provider>)

  return (
    `<!doctype html>
    <html>
      <head>
        <title>FIX ME</title>
        <link rel="stylesheet" href="${STATIC_PATH}/css/style.css">
      </head>
      <body>
        <div class="${APP_CONTAINER_CLASS}">${appHtml}</div>
        <script>
          window.__PRELOADED_STATE__ = ${JSON.stringify(store.getState())}
        </script>
        <script src="${isProd ? STATIC_PATH : `http://localhost:${WDS_PORT}/dist`}/js/bundle.js"></script>
      </body>
    </html>`
  )
}

export default renderApp
```

`ReactDOMServer.renderToString` 就是魔法發生的地方了。 React 會執行整個 `share` `App` 然後回傳一段包含 HTML 元素的字串。 `Provider` 與用戶端的運作模式相同，但在伺服器端，我們將應用包裹進 `StaticRouter` 而不是 `BrowserRouter`。為了讓 Redux `store` 的值從伺服器端傳遞到欲戶端，我們把他存進 `window.__PRELOADED_STATE__`，這是一個隨意命名的變數。

**備註**： `Immutable` 物件實作了 `toJSON()` 方法，也就是說你可以用 `JSON.stringify` 來將他們變成 JSON 字串

- 編輯 `scr/client/index.jsx` 來使用預載入的 `state` ：

```js
import * as Immutable from 'immutable'
// [...]

/* eslint-disable no-underscore-dangle */
const composeEnhancers = (isProd ? null : window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__) || compose
const preloadedState = window.__PRELOADED_STATE__
/* eslint-enable no-underscore-dangle */

const store = createStore(combineReducers(
  { hello: helloReducer }),
  { hello: Immutable.fromJS(preloadedState.hello) },
  composeEnhancers(applyMiddleware(thunkMiddleware)))
```

這裡我們將從伺服器端收到的 `preloadedState` 餵進用戶端的 `store`

🏁 你現在可以執行 `yarn start` 以及 `yarn dev:wds` 並且在頁面之間切換。重新整理這些頁面應該可以正確運作。請注意到 `message` 以及 `messageAsync` 會看起來不同，端看你是從用戶端瀏覽過去，還是是直接由伺服器端呈現。

### React Helmet

> 💡 **[React Helmet](https://github.com/nfl/react-helmet)**: 一個替 React 應用將內容注入到 `head` 部分的函式庫。在用戶端或伺服器端都可以用。

我故意在標題請你寫 `FIX ME` ，為了強調即使我們已經在做伺服器端長線，但我沒還沒將 `title` 正確地填寫（或者任何在 `head` 裡會出現的標籤，視不同頁面的需要。）

- 執行 `yarn add react-helmet`

- 編輯 `src/server/render-app.jsx` 如下：

```js
import Helmet from 'react-helmet'
// [...]
const renderApp = (/* [...] */) => {

  const appHtml = ReactDOMServer.renderToString(/* [...] */)
  const head = Helmet.rewind()

  return (
    `<!doctype html>
    <html>
      <head>
        ${head.title}
        ${head.meta}
        <link rel="stylesheet" href="${STATIC_PATH}/css/style.css">
      </head>
    [...]
    `
  )
}
```


React Helmet 使用 [react-side-effect](https://github.com/gaearon/react-side-effect) 的 `rewind` 來抓取一些我們的應用呈現的資料，很快我們就會呈現一些 `<Helmet />` 組件。這些 `<Helmet />` 組件就是我們要設定每一頁 `title` 以及其他 `head` 細節的地方。

- 編輯 `src/shared/app.jsx` 如下：

```js
import Helmet from 'react-helmet'
// [...]
const App = () =>
  <div>
    <Helmet titleTemplate={`%s | ${APP_NAME}`} defaultTitle={APP_NAME} />
    <Nav />
    // [...]
```

- 編輯 `src/shared/component/page/home.jsx` 如下：

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import { APP_NAME } from '../../config'

const HomePage = () =>
  <div>
    <Helmet
      meta={[
        { name: 'description', content: 'Hello App is an app to say hello' },
        { property: 'og:title', content: APP_NAME },
      ]}
    />
    <h1>{APP_NAME}</h1>
  </div>

export default HomePage

```

- 編輯 `src/shared/component/page/hello.jsx` 如下:

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import HelloButton from '../../container/hello-button'
import Message from '../../container/message'

const title = 'Hello Page'

const HelloPage = () =>
  <div>
    <Helmet
      title={title}
      meta={[
        { name: 'description', content: 'A page to say hello' },
        { property: 'og:title', content: title },
      ]}
    />
    <h1>{title}</h1>
    <Message />
    <HelloButton />
  </div>

export default HelloPage
```

- 編輯 `src/shared/component/page/hello-async.jsx` 如下:

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import HelloAsyncButton from '../../container/hello-async-button'
import MessageAsync from '../../container/message-async'

const title = 'Async Hello Page'

const HelloAsyncPage = () =>
  <div>
    <Helmet
      title={title}
      meta={[
        { name: 'description', content: 'A page to say hello asynchronously' },
        { property: 'og:title', content: title },
      ]}
    />
    <h1>{title}</h1>
    <MessageAsync />
    <HelloAsyncButton />
  </div>

export default HelloAsyncPage

```

- 編輯 `src/shared/component/page/not-found.jsx` 如下：

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

const title = 'Page Not Found'

const NotFoundPage = () =>
  <div>
    <Helmet
      title={title}
      meta={[
        { name: 'description', content: 'A page to say hello' },
        { property: 'og:title', content: title },
      ]}
    />
    <h1>{title}</h1>
  </div>

export default NotFoundPage
```


`<Helmet>` 組件沒有真的呈現任何東西，他只是將內容注入到文件中的 `head` 裡，並且對伺服器端開放同樣的資料。

🏁 執行 `yarn start` 以及 `yarn dev:wds` 並在頁面之中瀏覽。分頁上的標題文字應該會隨著你的瀏覽而改變，而當你重新整理的時候應該會保持一樣的內容。顯示你頁面的來源，來觀察 Helmet 是怎麼在伺服器端呈現裡更動 `title` 以及 `meta` 標籤。

下一章： [07 - Socket.IO](07-socket-io.md#readme)

回到 [上一張](05-redux-immutable-fetch.md#readme) 或 [內容目錄](https://github.com/verekia/js-stack-from-scratch#table-of-contents).
