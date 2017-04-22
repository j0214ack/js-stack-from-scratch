# 08 - Bootstrap and JSS

Code for this chapter available in the [`master-no-services`](https://github.com/verekia/js-stack-boilerplate/tree/master-no-services) branch of the [JS-Stack-Boilerplate repository](https://github.com/verekia/js-stack-boilerplate).

好吧！該給我們的醜陋的應用程序一個整容的時候了。我們將使用Twitter Bootstrap來給它一些基本樣式。然後我們將添加一個CSS-in-JS庫來添加一些自定義樣式。

## Twitter Bootstrap

> 💡 **[Twitter Bootstrap](http://getbootstrap.com/)** 是一個UI組件的函式庫

在React App 內，有兩個選項可以整合 Bootstrap。兩者皆各有優缺點：

- 使用官方釋出的版本，兩者皆在組件之下使用**jQuery**及**Tether**內。
- 使用第三方函式庫能在React內重構所有Bootstrap的組件，譬如[React-Bootstrap](https://react-bootstrap.github.io/)或[Reactstrap](https://reactstrap.github.io/)。

第三方函式庫提供非常方便React組件，與官方HTML
組件相比，大大減少了代碼膨脹，而且能與您的程式碼完全性的整合。
話雖如此，我必須說，我不願意使用它們，因為他們總釋出在官方版本之後（有時可能遠遠落後）。他們也不會使用Bootstrap主題來實現自己的JS。這是一個相當嚴重的缺失，考慮到Bootstrap的一個主要優勢是其龐大的設計師社區，使美麗的主題。

因此，我要作出衡量整合官方釋出的部份，以及jQuery和Tether。
我們關注的一個方法，關係到這個課程的打包大小。僅供參考，內含jQuery、Tether跟Bootstrap，打包大小約200kb。
我認為這是合理的，但是如果這對你還說還是太大，你應概略的考慮個除了Bootstrap的其他選擇，或者甚至不使用Bootstrap

### Bootstrap's CSS

- 刪除 `public/css/style.css`

- 從Bootstrap下載最新的官方釋出，並且將`bootstrap.min.css`及`bootstrap.min.css.map`放在`public/css`資料夾內。

- 編輯 `src/server/render-app.jsx`如下：

```html
<link rel="stylesheet" href="${STATIC_PATH}/css/bootstrap.min.css">
```

### Bootstrap's JS with jQuery and Tether

現在我們使用Bootstrap樣式套用在我們的頁面，我們必須將Javascript行為組件化。

- 執行 `yarn add jquery tether bootstrap@4.0.0-alpha.6`

- 編輯 `src/client/index.jsx` 如下:

```js
import $ from 'jquery'
import Tether from 'tether'

// [right after all your imports]

window.jQuery = $
window.Tether = Tether
require('bootstrap')
```

即將下載 Bootstrap's JavaScript 程式碼.

### Bootstrap 元件

好吧，到了複製貼上文件的時候了。

- 編輯 `src/shared/component/page/hello-async.jsx` 如下：

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import MessageAsync from '../../container/message-async'
import HelloAsyncButton from '../../container/hello-async-button'

const title = 'Async Hello Page'

const HelloAsyncPage = () =>
  <div className="container mt-4">
    <Helmet
      title={title}
      meta={[
        { name: 'description', content: 'A page to say hello asynchronously' },
        { property: 'og:title', content: title },
      ]}
    />
    <div className="row">
      <div className="col-12">
        <h1>{title}</h1>
        <MessageAsync />
        <HelloAsyncButton />
      </div>
    </div>
  </div>

export default HelloAsyncPage
```

- 編輯 `src/shared/component/page/hello.jsx` 如下：

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import Message from '../../container/message'
import HelloButton from '../../container/hello-button'

const title = 'Hello Page'

const HelloPage = () =>
  <div className="container mt-4">
    <Helmet
      title={title}
      meta={[
        { name: 'description', content: 'A page to say hello' },
        { property: 'og:title', content: title },
      ]}
    />
    <div className="row">
      <div className="col-12">
        <h1>{title}</h1>
        <Message />
        <HelloButton />
      </div>
    </div>
  </div>

export default HelloPage
```

- 編輯 `src/shared/component/page/home.jsx` 如下：

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import ModalExample from '../modal-example'
import { APP_NAME } from '../../config'

const HomePage = () =>
  <div>
    <Helmet
      meta={[
        { name: 'description', content: 'Hello App is an app to say hello' },
        { property: 'og:title', content: APP_NAME },
      ]}
    />
    <div className="jumbotron">
      <div className="container">
        <h1 className="display-3 mb-4">{APP_NAME}</h1>
      </div>
    </div>
    <div className="container">
      <div className="row">
        <div className="col-md-4 mb-4">
          <h3 className="mb-3">Bootstrap</h3>
          <p>
            <button type="button" role="button" data-toggle="modal" data-target=".js-modal-example" className="btn btn-primary">Open Modal</button>
          </p>
        </div>
        <div className="col-md-4 mb-4">
          <h3 className="mb-3">JSS (soon)</h3>
        </div>
        <div className="col-md-4 mb-4">
          <h3 className="mb-3">Websockets</h3>
          <p>Open your browser console.</p>
        </div>
      </div>
    </div>
    <ModalExample />
  </div>

export default HomePage
```

- 編輯 `src/shared/component/page/not-found.jsx` 如下：

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'
import { Link } from 'react-router-dom'
import { HOME_PAGE_ROUTE } from '../../routes'

const title = 'Page Not Found!'

const NotFoundPage = () =>
  <div className="container mt-4">
    <Helmet title={title} />
    <div className="row">
      <div className="col-12">
        <h1>{title}</h1>
        <div><Link to={HOME_PAGE_ROUTE}>Go to the homepage</Link>.</div>
      </div>
    </div>
  </div>

export default NotFoundPage
```

- 編輯 `src/shared/component/button.jsx` 如下：

```js
// [...]
<button
  onClick={handleClick}
  className="btn btn-primary"
  type="button"
  role="button"
>{label}</button>
// [...]
```

- 建立文件： `src/shared/component/footer.jsx`

```js
// @flow

import React from 'react'
import { APP_NAME } from '../config'

const Footer = () =>
  <div className="container mt-5">
    <hr />
    <footer>
      <p>© {APP_NAME} 2017</p>
    </footer>
  </div>

export default Footer
```

- 建立文件： `src/shared/component/modal-example.jsx`

```js
// @flow

import React from 'react'

const ModalExample = () =>
  <div className="js-modal-example modal fade">
    <div className="modal-dialog">
      <div className="modal-content">
        <div className="modal-header">
          <h5 className="modal-title">Modal title</h5>
          <button type="button" className="close" data-dismiss="modal">×</button>
        </div>
        <div className="modal-body">
          This is a Bootstrap modal. It uses jQuery.
        </div>
        <div className="modal-footer">
          <button type="button" role="button" className="btn btn-primary" data-dismiss="modal">Close</button>
        </div>
      </div>
    </div>
  </div>

export default ModalExample
```

- 編輯 `src/shared/app.jsx` 如下：

```js
const App = () =>
  <div style={{ paddingTop: 54 }}>
```

這是一個 *React的行內樣式範例*。

這段code將轉譯為: `<div style="padding-top:54px;">` 在你的dom物件內。
重點在於，我們需要這種樣式來讓內容在導覽列下。[React的行內樣式](https://speakerdeck.com/vjeux/react-css-in-js) 是最棒的，這種方式可以隔離全局css樣式與組件樣式，但他是個代價：你無法使用某些css特點，譬如 `:hover`、Media Queries、動畫、`font-face`。這是就為什麼本章節稍後提到要整合 CSS-in-JS library或JSS的原因。現階段，只要記住能使用React的行內樣式這個方法，假使你不需要使用`:hover`之類的。

- 編輯 `src/shared/component/nav.jsx` 如下：

```js
// @flow

import $ from 'jquery'
import React from 'react'
import { Link, NavLink } from 'react-router-dom'
import { APP_NAME } from '../config'
import {
  HOME_PAGE_ROUTE,
  HELLO_PAGE_ROUTE,
  HELLO_ASYNC_PAGE_ROUTE,
  NOT_FOUND_DEMO_PAGE_ROUTE,
} from '../routes'

const handleNavLinkClick = () => {
  $('body').scrollTop(0)
  $('.js-navbar-collapse').collapse('hide')
}

const Nav = () =>
  <nav className="navbar navbar-toggleable-md navbar-inverse fixed-top bg-inverse">
    <button className="navbar-toggler navbar-toggler-right" type="button" role="button" data-toggle="collapse" data-target=".js-navbar-collapse">
      <span className="navbar-toggler-icon" />
    </button>
    <Link to={HOME_PAGE_ROUTE} className="navbar-brand">{APP_NAME}</Link>
    <div className="js-navbar-collapse collapse navbar-collapse">
      <ul className="navbar-nav mr-auto">
        {[
          { route: HOME_PAGE_ROUTE, label: 'Home' },
          { route: HELLO_PAGE_ROUTE, label: 'Say Hello' },
          { route: HELLO_ASYNC_PAGE_ROUTE, label: 'Say Hello Asynchronously' },
          { route: NOT_FOUND_DEMO_PAGE_ROUTE, label: '404 Demo' },
        ].map(link => (
          <li className="nav-item" key={link.route}>
            <NavLink to={link.route} className="nav-link" activeStyle={{ color: 'white' }} exact onClick={handleNavLinkClick}>{link.label}</NavLink>
          </li>
        ))}
      </ul>
    </div>
  </nav>

export default Nav
```

這裡有個新東西稱作 `handleNavLinkClick` 。與一個問題相關，SPA使用 Bootstrap 的 `navbar` ，點擊手機上的連結無法收合選單，也無法點擊回頁首，這是個能展現 jQuery / Bootstrap-specific 程式碼整合在你的app範例內的好機會：

```js
import $ from 'jquery'
// [...]

const handleNavLinkClick = () => {
  $('body').scrollTop(0)
  $('.js-navbar-collapse').collapse('hide')
}

<NavLink /* [...] */ onClick={handleNavLinkClick}>
```

**筆記**：我已經移除了無障礙相關的屬性（譬如 `aria` 屬性）使程式碼在這份教學文件的上下文內更易讀，**你應該把他們放回來**，並參閱Bootstrap的文件和程式碼例子，以了解該如何使用。

🏁 您的app設計現在應該完全如同 Bootstrap。

## CSS的當前狀態

在2016年，典型的現代JavaScript 程式解法。本教學為您設置的不同的函式庫和工具幾乎是最尖端的業界標準（*即使它可能在一年後完全過時*），是的，這是個複雜的程式碼設定，但至少，大多數的前端開發者同意React-Redux-Webpack是個辦法。現在關於css有個壞消息，沒有標準的方式、也沒有標準的解法。

SASS, BEM, SMACSS, SUIT, Bass CSS, React行內樣式， LESS, Styled Components, CSSX, JSS, Radium, Web Components, CSS Modules, OOCSS, Tachyons, Stylus, Atomic CSS, PostCSS, Aphrodite, React Native 使用於網站的，還有許多完成工作的不同方法或工具，是我忘記的。都做得很好，但這就是個大問題，因為沒有明確的勝出者。

現在比較酷的作法，像是喜歡使用 React內聯樣式，CSS-in-JS或CSS模塊的方法，因為它們與React非常好地整合，並以編程方式解決了常規CSS方法的[問題](https://speakerdeck.com/vjeux/react-css-in-js) 。

css模組化工作更棒，因為但是它們並沒有利用JavaScript的強大功能和CSS上的許多功能，他們只提供封裝，這是很好的，但是React的行內樣式和CSS-in-JS 在我看來，是樣式設計到另一個層次。我個人的建議是，React的行內樣式，應該使用於一般常用的樣式（這就是你必須使用於React Native），並且使用一個 CSS-in-JS 函式庫，就像 `:hover` 與 media queries。

這就是CSS-in-JS函式庫的調性，兩種領先的分別是Aphrodite跟JSS。他們實現了相當多同樣的東西，連語法基本上也差不多。老實說，我沒有對任何大型專案進行比較，而且我真的比較偏愛JSS's API。我們也可以對這個話題來討論，我想聽看看那些做得比較徹底的人的 [意見](https://github.com/verekia/js-stack-from-scratch/issues/139)。

## JSS

> 💡 **[JSS](http://cssinjs.org/)** 是一種可以把css樣式寫在 javascript 內的作法，是一種CSS-in-JS的函式庫，應用在產品內。

現在我們有一些Bootstrap基本的模版，我們來寫一些自定義的css樣式。剛才提到React行內樣式無法控制譬如 `:hover`與 media queries，所以我們使用JSS來展現一個簡單的網頁範例，JSS透過使用 `react-jss`，一個便於與React組件共同使用的函式庫。

- 執行 `yarn add react-jss`

將以下內容添加到 `.flowconfig` 文件中，因為目前JSS有一個Flow [問題](https://github.com/cssinjs/jss/issues/411)：

```flowconfig
[ignore]
.*/node_modules/jss/.*
```

### Server 端

JSS可以在服務器上進行初始渲染以呈現樣式。

- 將以下常數添加到 `src/shared/config.js`:

```js
export const JSS_SSR_CLASS = 'jss-ssr'
export const JSS_SSR_SELECTOR = `.${JSS_SSR_CLASS}`
```

- 編輯 `src/server/render-app.jsx` 如下：

```js
import { SheetsRegistry, SheetsRegistryProvider } from 'react-jss'
// [...]
import { APP_CONTAINER_CLASS, JSS_SSR_CLASS, STATIC_PATH, WDS_PORT } from '../shared/config'
// [...]
const renderApp = (location: string, plainPartialState: ?Object, routerContext: ?Object = {}) => {
  const store = initStore(plainPartialState)
  const sheets = new SheetsRegistry()
  const appHtml = ReactDOMServer.renderToString(
    <Provider store={store}>
      <StaticRouter location={location} context={routerContext}>
        <SheetsRegistryProvider registry={sheets}>
          <App />
        </SheetsRegistryProvider>
      </StaticRouter>
    </Provider>)
  // [...]
      <link rel="stylesheet" href="${STATIC_PATH}/css/bootstrap.min.css">
      <style class="${JSS_SSR_CLASS}">${sheets.toString()}</style>
  // [...]
```

## Client 端

客戶端應該做的第一件事，是渲染應用程式客戶端後，擺脫服務器生成的JSS樣式。

- 在調用 `ReactDOM.render`之後（例如在`setUpSocket(store)`之前），將以下內容增加到 `src/client/index.jsx` 中）：

```js
import { APP_CONTAINER_SELECTOR, JSS_SSR_SELECTOR } from '../shared/config'
// [...]

const jssServerSide = document.querySelector(JSS_SSR_SELECTOR)
// flow-disable-next-line
jssServerSide.parentNode.removeChild(jssServerSide)

setUpSocket(store)
```

- 編輯 `src/shared/component/page/home.jsx` 如下:

```js
import injectSheet from 'react-jss'
// [...]
const styles = {
  hoverMe: {
    '&:hover': {
      color: 'red',
    },
  },
  '@media (max-width: 800px)': {
    resizeMe: {
      color: 'red',
    },
  },
}

const HomePage = ({ classes }: { classes: Object }) =>
  // [...]
  <div className="col-md-4 mb-4">
    <h3 className="mb-3">JSS</h3>
    <p className={classes.hoverMe}>Hover me.</p>
    <p className={classes.resizeMe}>Resize the window.</p>
  </div>
  // [...]

export default injectSheet(styles)(HomePage)
```

與React的行內樣式不同，JSS使用傳統樣式。您將樣式傳遞給 `injectSheet`，CSS樣式最終會在您的元件內。

🏁 執行 `yarn start` 及 `yarn dev:wds`。打開網頁，看裡面的原始碼（不是檢查樣式）看到JSS樣式存在於DOM中再一開始在 `<style class="jss-ssr">` 內渲染出的元素（只有這個網頁內的）。他們應該去檢查、替代為 `<style type="text/css" data-jss data-meta="HomePage">`。

**筆記**: 在生產模式下， `data-meta` 易混淆。貼心！

如果您將鼠標懸停在"Hover me"標籤上，應變為紅色。如果您將瀏覽器寬度調整為小於800px，則"Resize your window"標籤應變為紅色。

下一章節: [09 - Travis, Coveralls, Heroku](09-travis-coveralls-heroku.md#readme)

Back to the [previous section](07-socket-io.md#readme) or the [table of contents](https://github.com/verekia/js-stack-from-scratch#table-of-contents).
