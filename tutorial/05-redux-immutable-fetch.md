# 05 - Redux, Immutable, and Fetch

本章節的程式碼 [點此](https://github.com/verekia/js-stack-walkthrough/tree/master/05-redux-immutable-fetch).

在這個章節，我們將會使用 React 和 Redux 來製作一個簡單的應用程式。這個應用程式會包含一則訊息和一個按鈕。當按下按鈕時，訊息內容便會改變。
In this chapter we will hook up React and Redux to make a very simple app. The app will consist of a message and a button. The message changes when the user clicks the button.

開始之前，讓我們先認識 ImmutableJS。雖然與 React 和 Redux 無關，但我們會在這個章節使用到。
Before we start, here is a very quick introduction to ImmutableJS, which is completely unrelated to React and Redux, but will be used in this chapter.

## ImmutableJS

> 💡 **[ImmutableJS](https://facebook.github.io/immutable-js/)** (或稱作 Immutable) 是一個由 Facebook 開發的函式庫，功能是操作不可變的集合，如 lists 和 maps。（譯註：array 轉換為 list，object 轉換為 map。）任何意圖使 immutable 物件改變都會回傳一個新的物件，而不會改變原先的物件。Any change made on an immutable object returns a new object without mutating the original object.
> 💡 **[ImmutableJS](https://facebook.github.io/immutable-js/)** (or just Immutable) is a library by Facebook to manipulate immutable collections, like lists and maps. Any change made on an immutable object returns a new object without mutating the original object.

舉例，我們不會這麼做：
For instance, instead of doing:

```js
const obj = { a: 1 }
obj.a = 2 // 改變了 `obj` Mutates `obj`
```

而是會：
You would do:

```js
const obj = Immutable.Map({ a: 1 })
obj.set('a', 2) // 回傳一個新的物件，而不改變 `obj` Returns a new object without mutating `obj`
```

這項工具遵從了 **功能性編程（functional programming）** 的思維方式而能與 Redux 合作無間。
This approach follows the **functional programming** paradigm, which works really well with Redux.

當創造 immutable 物件時，最方便的方法是 `Immutable.fromJS()`。這項方法可用在物件或是陣列上，回傳不可變的相同版本
When creating immutable collections, a very convenient method is `Immutable.fromJS()`, which takes any regular JS object or array and returns a deeply immutable version of it:

```js
const immutablePerson = Immutable.fromJS({
  name: 'Stan',
  friends: ['Kyle', 'Cartman', 'Kenny'],
})

console.log(immutablePerson)

/*
 *  Map {
 *    "name": "Stan",
 *    "friends": List [ "Kyle", "Cartman", "Kenny" ]
 *  }
 */
```

- 執行 `yarn add immutable`
- Run `yarn add immutable`

**注意**: 由於 ImmutableJS 的輸出方式，以 `import Immutable from 'immutable'` 引入會產生錯誤，改用 `import * as Immutable from 'immutable'` 替代並祈禱早日[修復](https://github.com/facebook/immutable-js/issues/863)。
**Note**: Due to the implementation of ImmutableJS, Flow does not accept importing it with `import Immutable from 'immutable'`, so use this syntax instead: `import * as Immutable from 'immutable'`. Let's cross fingers for a [fix](https://github.com/facebook/immutable-js/issues/863) soon.

## Redux

> 💡 **[Redux](http://redux.js.org/)** 是一個函式庫，用來處理應用程式的生命週期。 Redux 藉由建立唯一的 *store* 來存放所有的 state。
> 💡 **[Redux](http://redux.js.org/)** is a library to handle the lifecycle of your application. It creates a *store*, which is the single source of truth of the state of your app at any given time.

我們先從簡單的部分下手，宣告我們的 Redux actions：
Let's start with the easy part, declaring our Redux actions:

- 執行 `yarn add redux redux-actions`
- Run `yarn add redux redux-actions`

- 新增 `src/client/action/hello.js` 內容包含：
- Create a `src/client/action/hello.js` file containing:

```js
// @flow

import { createAction } from 'redux-actions'

export const SAY_HELLO = 'SAY_HELLO'

export const sayHello = createAction(SAY_HELLO)
```

這個檔案輸出一個 *action*： `SAY_HELLO` 和 *action 創建函式（creator）*： `sayHello`。我們使用[`redux-actions`](https://github.com/acdlite/redux-actions) 來減少與 Redux actions 相關的重複性程式碼。 `redux-actions` 實踐了 [Flux Standard Action](https://github.com/acdlite/flux-standard-action) 模型，使 *action 創建函式* 回傳具有 `type` 和 `payload` 屬性的物件
This file exposes an *action*, `SAY_HELLO`, and its *action creator*, `sayHello`, which is a function. We use [`redux-actions`](https://github.com/acdlite/redux-actions) to reduce the boilerplate associated with Redux actions. `redux-actions` implement the [Flux Standard Action](https://github.com/acdlite/flux-standard-action) model, which makes *action creators* return objects with the `type` and `payload` attributes.

- 新增 `src/client/reducer/hello.js` 內容包含：
- Create a `src/client/reducer/hello.js` file containing:

```js
// @flow

import * as Immutable from 'immutable'

import { SAY_HELLO } from '../action/hello'

const initialState = Immutable.fromJS({
  message: '初始 reducer 訊息',
})

const helloReducer = (state: Object = initialState, action: { type: string, payload: any }) => {
  switch (action.type) {
    case SAY_HELLO:
      return state.set('message', action.payload)
    default:
      return state
  }
}

export default helloReducer
```

在這個檔案，我們以 Immutable Map 初始化了 reducer 的 state，並設定 `message` 內容為 `Initial reducer message`。Flow annotation 解構 `action` 為 `type` 和 `payload`。`type` 為字串，`payload` 可以是 `any` 型別。`helloReducer` 一旦判定 `type` 符合 `SAY_HELLO` ，便將 action payload 設為新的 `message` 內容。如果你未曾接觸過，這看得來可能會有點新奇古怪，不過大致還是可以理解的。特別注意到我們前段學過 `Immutable.fromJS()` 和 `set()` 的用法。
In this file we initialize the state of our reducer with an Immutable Map containing one property, `message`, set to `Initial reducer message`. The `helloReducer` handles `SAY_HELLO` actions by simply setting the new `message` with the action payload. The Flow annotation for `action` destructures it into a `type` and a `payload`. The `payload` can be of `any` type. It looks funky if you've never seen this before, but it remains pretty understandable. Note the usage of `Immutable.fromJS()` and `set()` as seen before.

## React-Redux

> 💡 **[react-redux](https://github.com/reactjs/react-redux)** *組合* Redux store 和 React 組件。 有了 `react-redux`，當 Redux store 改變時，React components 便會自動更新，另外也可以產出 Redux actions。
> 💡 **[react-redux](https://github.com/reactjs/react-redux)** *connects* a Redux store with React components. With `react-redux`, when the Redux store changes, React components get automatically updated. They can also fire Redux actions.

- 執行 `yarn add react-redux`
- Run `yarn add react-redux`

在這個段落，我們會建立 *Components* 和 *Containers*。
In this section we are going to create *Components* and *Containers*.

**Components** 是 *好笨笨* React 組件，也就是它們與 Redux state 無關。**Containers** 是 *好棒棒* React 組件，與 state 有關係並 *連結* 我們的好笨笨組件。
**Components** are *dumb* React components, in a sense that they don't know anything about the Redux state. **Containers** are *smart* components that know about the state and that we are going to *connect* to our dumb components.

- 新增 `src/client/component/button.jsx` 內容包含：
- Create a `src/client/component/button.jsx` file containing:

```js
// @flow

import React, { PropTypes } from 'react'

const Button = ({ label, handleClick }: { label: string, handleClick: Function }) =>
  <button onClick={handleClick}>{label}</button>

Button.propTypes = {
  label: PropTypes.string.isRequired,
  handleClick: PropTypes.func.isRequired,
}

export default Button
```
**注意**： 你可以看到另一個 Flow annotation 解構的範例。如果 `props` 包含 `handleClick`，我們會寫 `const Button = ({ handleClick }: { handleClick: Function }) => { handleClick() }` 來取代 `const Button = (props) => { props.handleClick() }`。這語法雖然有點笨拙，但很值得。
**Note**: You can see another case of destructuring with Flow annotations here. If `props` contains `handleClick`, instead of writing `const Button = (props) => { props.handleClick() }`, we write `const Button = ({ handleClick }: { handleClick: Function }) => { handleClick() }`. The syntax is a bit cumbersome but worth it.

- 新增 `src/client/component/message.jsx` 內容包含：
- Create a `src/client/component/message.jsx` file containing:

```js
// @flow

import React, { PropTypes } from 'react'

const Message = ({ message }: { message: string }) =>
  <p>{message}</p>

Message.propTypes = {
  message: PropTypes.string.isRequired,
}

export default Message
```

上述是 *好笨笨* 組件的範例。不需邏輯，顯示任何透過 React **props** 傳入的東西。`button.jsx` 和 `message.jsx` 最主要的差別在於 `Button` 包含了action dispatcher 在 props 當中，而 `Message` 只是包含要呈現的資料。
These are examples of *dumb* components. They are logic-less, and just show whatever they are asked to show via React **props**. The main difference between `button.jsx` and `message.jsx` is that `Button` contains a reference to an action dispatcher in its props, where `Message` just contains some data to show.

再來一次，*components* 與 Redux **actions** 或 **state** 都無關係，因此我們需要建立 **containers** 把適當的 action dispatchers 和資料給上面兩個好笨笨組件。
 *components* don't know anything about Redux **actions** or the **state** of our app, which is why we are going to create smart **containers** that will feed the proper action dispatchers and data to these 2 dumb components.

- 建立 `src/client/container/hello-button.js` 內容包含：
- Create a `src/client/container/hello-button.js` file containing:

```js
// @flow

import { connect } from 'react-redux'

import { sayHello } from '../action/hello'
import Button from '../component/button'

const mapStateToProps = () => ({
  label: 'Say hello',
})

const mapDispatchToProps = dispatch => ({
  handleClick: () => { dispatch(sayHello('Hello!')) },
})

export default connect(mapStateToProps, mapDispatchToProps)(Button)
```

這個 container 串起 `Button` 組件和 `sayHello` action 以及 Redux's `dispatch` 函式。
This container hooks up the `Button` component with the `sayHello` action and Redux's `dispatch` method.

- 建立 `src/client/container/message.js` 內容包含：

```js
// @flow

import { connect } from 'react-redux'

import Message from '../component/message'

const mapStateToProps = state => ({
  message: state.hello.get('message'),
})

export default connect(mapStateToProps)(Message)
```

這個 container 串起 state 和 `Message` 組件。當 state 改變，`Message` 會以新的`message` prop重新繪製。這些連結都是經由 `react-redux` 的 `connect` 函式完成。
This container hooks up the Redux's app state with the `Message` component. When the state changes, `Message` will now automatically re-render with the proper `message` prop. These connections are done via the `connect` function of `react-redux`.

- 更新 `src/client/app.jsx` 看起來會像：
- Update your `src/client/app.jsx` file like so:

```js
// @flow

import React from 'react'
import HelloButton from './container/hello-button'
import Message from './container/message'
import { APP_NAME } from '../shared/config'

const App = () =>
  <div>
    <h1>{APP_NAME}</h1>
    <Message />
    <HelloButton />
  </div>

export default App
```

我們還沒啟動 Redux store ，也還沒把兩個 containers 放進我們的程式裡：
We still haven't initialized the Redux store and haven't put the 2 containers anywhere in our app yet:

- 更新 `src/client/index.jsx` 看起來會像：

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'
import { AppContainer } from 'react-hot-loader'
import { Provider } from 'react-redux'
import { createStore, combineReducers } from 'redux'

import App from './app'
import helloReducer from './reducer/hello'
import { APP_CONTAINER_SELECTOR } from '../shared/config'
import { isProd } from '../shared/util'

const store = createStore(combineReducers({ hello: helloReducer }),
  // eslint-disable-next-line no-underscore-dangle
  isProd ? undefined : window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__())

const rootEl = document.querySelector(APP_CONTAINER_SELECTOR)

const wrapApp = (AppComponent, reduxStore) =>
  <Provider store={reduxStore}>
    <AppContainer>
      <AppComponent />
    </AppContainer>
  </Provider>

ReactDOM.render(wrapApp(App, store), rootEl)

if (module.hot) {
  // flow-disable-next-line
  module.hot.accept('./app', () => {
    // eslint-disable-next-line global-require
    const NextApp = require('./app').default
    ReactDOM.render(wrapApp(NextApp, store), rootEl)
  })
}
```

我們來檢視一下。首先，用 `createStore` 傳入 reducers 來建立 *store*。這裡我們只有一個 reducer，但為了以後也可以適用，我們使用 `combineReducers` 來組合所有 reducers。最後傳入 `createStore` 的古怪參數是串起 Redux 和瀏覽器 [開發工具](https://github.com/zalmoxisus/redux-devtools-extension)，這項工具在除錯時非常有用。因為 ESLint 會顯示 `__REDUX_DEVTOOLS_EXTENSION__` 的底線錯誤，我們必須關閉這項規則。 再來， 我們用 `wrapApp` 函式很方便地將我們的程式包在 `react-redux` 的 `Provider` 組件裡，同時傳入 store。
Let's take a moment to review this. First, we create a *store* with `createStore`. Stores are created by passing reducers to them. Here we only have one reducer, but for the sake of future scalability, we use `combineReducers` to group all of our reducers together. The last weird parameter of `createStore` is something to hook up Redux to browser [Devtools](https://github.com/zalmoxisus/redux-devtools-extension), which are incredibly useful when debugging. Since ESLint will complain about the underscores in `__REDUX_DEVTOOLS_EXTENSION__`, we disable this ESLint rule. Next, we conveniently wrap our entire app inside `react-redux`'s `Provider` component thanks to our `wrapApp` function, and pass our store to it.

🏁 現在你可以執行 `yarn start` 和 `yarn dev:wds`，然後看看 `http://localhost:8000`。你應該會看到 "初始 reducer 訊息" 和一個按鈕。當你按下按鈕，訊息應該會變為 "Hello!"。如果你有在你的瀏覽器安裝 Redux Devtools，當你按下按鈕，你會看見 state 的改變。
🏁 You can now run `yarn start` and `yarn dev:wds` and hit `http://localhost:8000`. You should see "Initial reducer message" and a button. When you click the button, the message should change to "Hello!". If you installed the Redux Devtools in your browser, you should see the app state change over time as you click on the button.

恭喜恭喜，我們終於完成一支會做一些事的程式了！好啦，雖然表面看起來不是那麼 *令人驚艷*，但我們都知道裡面有最潮的東西。
Congratulations, we finally made an app that does something! Okay it's not a *super* impressive from the outside, but we all know that it is powered by one badass stack under the hood.

## 用非同步請求延伸我們的程式
## Extending our app with an asynchronous call

我們現在要加入第二個按鈕，功用是發出 AJAX 請求到伺服器來獲取新的訊息。為了提供展示，這個請求同時會送出寫死的數字 `1234`。
We are now going to add a second button to our app, which will trigger an AJAX call to retrieve a message from the server. For the sake of demonstration, this call will also send some data, the hard-coded number `1234`.

### 伺服器端
### The server endpoint

- 建立 `src/shared/routes.js` 內容包含：

```js
// @flow

// eslint-disable-next-line import/prefer-default-export
export const helloEndpointRoute = (num: ?number) => `/ajax/hello/${num || ':num'}`
```

這個函式幫助產生下列的提示：
This function is a little helper to produce the following:

```js
helloEndpointRoute()     // -> '/ajax/hello/:num' (for Express)
helloEndpointRoute(1234) // -> '/ajax/hello/1234' (for the actual call)
```

我們來實際建立一個測試，來證實這會成功。
Let's actually create a test real quick to make sure this thing works well.

- 建立 `src/shared/routes.test.js` 內容包含：
- Create a `src/shared/routes.test.js` containing:

```js
import { helloEndpointRoute } from './routes'

test('helloEndpointRoute', () => {
  expect(helloEndpointRoute()).toBe('/ajax/hello/:num')
  expect(helloEndpointRoute(123)).toBe('/ajax/hello/123')
})
```

- 執行 `yarn test` 這應該會成功通過測試。
- Run `yarn test` and it should pass successfully.

- 在 `src/server/index.js` 加入下列：
- In `src/server/index.js`, add the following:

```js
import { helloEndpointRoute } from '../shared/routes'

// [under app.get('/')...]

app.get(helloEndpointRoute(), (req, res) => {
  res.json({ serverMessage: `Hello from the server! (received ${req.params.num})` })
})
```

### 新的 containers
### New containers

- 建立 `src/client/container/hello-async-button.js` 內容包含：
- Create a `src/client/container/hello-async-button.js` file containing:

```js
// @flow

import { connect } from 'react-redux'

import { sayHelloAsync } from '../action/hello'
import Button from '../component/button'

const mapStateToProps = () => ({
  label: 'Say hello asynchronously and send 1234',
})

const mapDispatchToProps = dispatch => ({
  handleClick: () => { dispatch(sayHelloAsync(1234)) },
})

export default connect(mapStateToProps, mapDispatchToProps)(Button)
```

為了要展示你如何傳入參數到非同步請求，並讓這件事很簡單。我將 `1234` 寫死。這個值通常來自使用者在表單填入的值。
In order to demonstrate how you would pass a parameter to your asynchronous call and to keep things simple, I am hard-coding a `1234` value here. This value would typically come from a form field filled by the user.

- 建立 `src/client/container/message-async.js` 內容包含：
- Create a `src/client/container/message-async.js` file containing:

```js
// @flow

import { connect } from 'react-redux'

import MessageAsync from '../component/message'

const mapStateToProps = state => ({
  message: state.hello.get('messageAsync'),
})

export default connect(mapStateToProps)(MessageAsync)
```

你可以在這個 container 看到我們參照 `messageAsync` 屬性，我們將在 reducer 添加這個屬性。
You can see that in this container, we are referring to a `messageAsync` property, which we're going to add to our reducer soon.

我們現在需要做的是建立 `sayHelloAsync` action.
What we need now is to create the `sayHelloAsync` action.

### Fetch

> 💡 **[Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)** 一個產生非同步請求的標準 JavaScript 函式，受到 jQuery's AJAX 函式啟發而來。
> 💡 **[Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)** is a standardized JavaScript function to make asynchronous calls inspired by jQuery's AJAX methods.

我們會使用 `fetch` 從用戶端來向伺服器端發出請求。`fetch` 尚未被所有瀏覽器支援，因此我們需要一個 polyfill。`isomorphic-fetch` 正是一個 polyfill 可以讓 `fetch` 在跨瀏覽器和 Node 運行無誤！
We are going to use `fetch` to make calls to the server from the client. `fetch` is not supported by all browsers yet, so we are going to need a polyfill. `isomorphic-fetch` is a polyfill that makes it work cross-browsers and in Node too!

- 執行 `yarn add isomorphic-fetch`
- Run `yarn add isomorphic-fetch`

### 三個非同步 actions
### 3 asynchronous actions

`sayHelloAsync` 不是一個常規的 action。非同步 actions 通常會拆成三個 actions 對應到三個不同狀態： *請求* action (或是 "讀取中")、*成功* action、*失敗* action。
`sayHelloAsync` is not going to be a regular action. Asynchronous actions are usually split into 3 actions, which trigger 3 different states: a *request* action (or "loading"), a *success* action, and a *failure* action.

- 更新 `src/client/action/hello.js` 看起來像：
- Edit `src/client/action/hello.js` like so:

```js
// @flow

import 'isomorphic-fetch'

import { createAction } from 'redux-actions'
import { helloEndpointRoute } from '../../shared/routes'

export const SAY_HELLO = 'SAY_HELLO'
export const SAY_HELLO_ASYNC_REQUEST = 'SAY_HELLO_ASYNC_REQUEST'
export const SAY_HELLO_ASYNC_SUCCESS = 'SAY_HELLO_ASYNC_SUCCESS'
export const SAY_HELLO_ASYNC_FAILURE = 'SAY_HELLO_ASYNC_FAILURE'

export const sayHello = createAction(SAY_HELLO)
export const sayHelloAsyncRequest = createAction(SAY_HELLO_ASYNC_REQUEST)
export const sayHelloAsyncSuccess = createAction(SAY_HELLO_ASYNC_SUCCESS)
export const sayHelloAsyncFailure = createAction(SAY_HELLO_ASYNC_FAILURE)

export const sayHelloAsync = (num: number) => (dispatch: Function) => {
  dispatch(sayHelloAsyncRequest())
  return fetch(helloEndpointRoute(num), { method: 'GET' })
    .then((res) => {
      if (!res.ok) throw Error(res.statusText)
      return res.json()
    })
    .then((data) => {
      if (!data.serverMessage) throw Error('No message received')
      dispatch(sayHelloAsyncSuccess(data.serverMessage))
    })
    .catch(() => {
      dispatch(sayHelloAsyncFailure())
    })
}
```

`sayHelloAsync` 不會回傳 action，而是回傳一個會發出 `fetch` 請求的函式。而 `fetch` 回傳 `Promise`，依據非同步請求的不同狀態，我們使用 `Promise` 來 *做出* 不同行動。
Instead of returning an action, `sayHelloAsync` returns a function which launches the `fetch` call. `fetch` returns a `Promise`, which we use to *dispatch* different actions depending on the current state of our asynchronous call.

### 三個非同步 action 處理器
### 3 asynchronous action handlers

我們來處理這些在 `src/client/reducer/hello.js` 的不同 actions：
Let's handle these different actions in `src/client/reducer/hello.js`:

```js
// @flow

import * as Immutable from 'immutable'

import {
  SAY_HELLO,
  SAY_HELLO_ASYNC_REQUEST,
  SAY_HELLO_ASYNC_SUCCESS,
  SAY_HELLO_ASYNC_FAILURE,
} from '../action/hello'

const initialState = Immutable.fromJS({
  message: '初始 reducer 訊息',
  messageAsync: '初始 reducer 非同步請求訊息',
})

const helloReducer = (state: Object = initialState, action: { type: string, payload: any }) => {
  switch (action.type) {
    case SAY_HELLO:
      return state.set('message', action.payload)
    case SAY_HELLO_ASYNC_REQUEST:
      return state.set('messageAsync', '讀取中...')
    case SAY_HELLO_ASYNC_SUCCESS:
      return state.set('messageAsync', action.payload)
    case SAY_HELLO_ASYNC_FAILURE:
      return state.set('messageAsync', '收不到訊息，檢查你的連線。')
    default:
      return state
  }
}

export default helloReducer
```

我們加入了不同欄位在 store 裡，`messageAsync` 會依據我們收到的不同 action 來更新。`SAY_HELLO_ASYNC_REQUEST` 會給出 `讀取中...` 訊息。`SAY_HELLO_ASYNC_SUCCESS` 會更新 `messageAsync` 就像 `SAY_HELLO` 會更新 `message`。`SAY_HELLO_ASYNC_FAILURE` 會給出錯誤訊息。
We added a new field to our store, `messageAsync`, and we update it with different messages depending on the action we receive. During `SAY_HELLO_ASYNC_REQUEST`, we show `Loading...`. `SAY_HELLO_ASYNC_SUCCESS` updates `messageAsync` similarly to how `SAY_HELLO` updates `message`. `SAY_HELLO_ASYNC_FAILURE` gives an error message.

### Redux-thunk

在 `src/client/action/hello.js` 我們建立 `sayHelloAsync` 這個 action 創建函式去回傳一個函式。原生 Redux 並不接受 action 為一個函式，為了要使這些非同步 actions 運作，我們必須要使用 `redux-thunk` 這個 *middleware*。
In `src/client/action/hello.js`, we made `sayHelloAsync`, an action creator that returns a function. This is actually not a feature that is natively supported by Redux. In order to perform these async actions, we need to extend Redux's functionality with the `redux-thunk` *middleware*.

- 執行 `yarn add redux-thunk`
- Run `yarn add redux-thunk`

- 更新 `src/client/index.jsx` 看起來像：
- Update your `src/client/index.jsx` file like so:

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'
import { AppContainer } from 'react-hot-loader'
import { Provider } from 'react-redux'
import { createStore, combineReducers, applyMiddleware, compose } from 'redux'
import thunkMiddleware from 'redux-thunk'

import App from './app'
import helloReducer from './reducer/hello'
import { APP_CONTAINER_SELECTOR } from '../shared/config'
import { isProd } from '../shared/util'

// eslint-disable-next-line no-underscore-dangle
const composeEnhancers = (isProd ? null : window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__) || compose

const store = createStore(combineReducers({ hello: helloReducer }),
  composeEnhancers(applyMiddleware(thunkMiddleware)))

const rootEl = document.querySelector(APP_CONTAINER_SELECTOR)

const wrapApp = (AppComponent, reduxStore) =>
  <Provider store={reduxStore}>
    <AppContainer>
      <AppComponent />
    </AppContainer>
  </Provider>

ReactDOM.render(wrapApp(App, store), rootEl)

if (module.hot) {
  // flow-disable-next-line
  module.hot.accept('./app', () => {
    // eslint-disable-next-line global-require
    const NextApp = require('./app').default
    ReactDOM.render(wrapApp(NextApp, store), rootEl)
  })
}
```

這裡我們傳入 `redux-thunk` 到 Redux 的 `applyMiddleware` 函式。為了讓 Redux 開發工具也能運作，我們需要用到 Redux's `compose` 函式。不用太擔心這部分，只要知道我們用了 `redux-thunk` 來強化 Redux 就好。
Here we pass `redux-thunk` to Redux's `applyMiddleware` function. In order for the Redux Devtools to keep working, we also need to use Redux's `compose` function. Don't worry too much about this part, just remember that we enhance Redux with `redux-thunk`.

- 更新 `src/client/app.jsx` 看起來像：
- Update `src/client/app.jsx` like so:

```js
// @flow

import React from 'react'
import HelloButton from './container/hello-button'
import HelloAsyncButton from './container/hello-async-button'
import Message from './container/message'
import MessageAsync from './container/message-async'
import { APP_NAME } from '../shared/config'

const App = () =>
  <div>
    <h1>{APP_NAME}</h1>
    <Message />
    <HelloButton />
    <MessageAsync />
    <HelloAsyncButton />
  </div>

export default App
```

🏁 執行 `yarn start` 和 `yarn dev:wds`。你應該可以按下 "Say hello asynchronously and send 1234" 然後從伺服器收到訊息！由於你是在本機作業，這個請求是非常即時的，但是看看 Redux 開發工具，你會看到每一次按下按鈕都會觸發 `SAY_HELLO_ASYNC_REQUEST` 和 `SAY_HELLO_ASYNC_SUCCESS`, 包含預料會看到的 `讀取中...` 訊息。
🏁 Run `yarn start` and `yarn dev:wds` and you should now be able to click the "Say hello asynchronously and send 1234" button and retrieve a corresponding message from the server! Since you're working locally, the call is instantaneous, but if you open the Redux Devtools, you will notice that each click triggers both `SAY_HELLO_ASYNC_REQUEST` and `SAY_HELLO_ASYNC_SUCCESS`, making the message go through the intermediate `Loading...` state as expected.

恭喜你自己了，這是很有深度的一章！我們來進行測試並完成這章吧。
You can congratulate yourself, that was an intense section! Let's wrap it up with some testing.

## 測試
## Testing

這個段落，要測試我們的 actions 和 reducer。先從 actions 開始吧。
In this section, we are going to test our actions and reducer. Let's start with the actions.

為了讓 `action/hello.js` 的概念獨立出來，我們需要 *假裝* 已經完成了不相關的部分，同時也要假裝 `fetch` 的 AJAX 請求有正確運作。
In order to isolate the logic that is specific to `action/hello.js` we are going to need to *mock* things that don't concern it, and also mock that AJAX `fetch` request which should not trigger an actual AJAX in our tests.

- 執行 `yarn add --dev redux-mock-store fetch-mock`
- Run `yarn add --dev redux-mock-store fetch-mock`

- 建立 `src/client/action/hello.test.js` 內容包含：
- Create a `src/client/action/hello.test.js` containing:

```js
import fetchMock from 'fetch-mock'
import configureMockStore from 'redux-mock-store'
import thunkMiddleware from 'redux-thunk'

import {
  sayHelloAsync,
  sayHelloAsyncRequest,
  sayHelloAsyncSuccess,
  sayHelloAsyncFailure,
} from './hello'

import { helloEndpointRoute } from '../../shared/routes'

const mockStore = configureMockStore([thunkMiddleware])

afterEach(() => {
  fetchMock.restore()
})

test('sayHelloAsync success', () => {
  fetchMock.get(helloEndpointRoute(666), { serverMessage: '非同步請求成功' })
  const store = mockStore()
  return store.dispatch(sayHelloAsync(666))
    .then(() => {
      expect(store.getActions()).toEqual([
        sayHelloAsyncRequest(),
        sayHelloAsyncSuccess('非同步請求成功'),
      ])
    })
})

test('sayHelloAsync 404', () => {
  fetchMock.get(helloEndpointRoute(666), 404)
  const store = mockStore()
  return store.dispatch(sayHelloAsync(666))
    .then(() => {
      expect(store.getActions()).toEqual([
        sayHelloAsyncRequest(),
        sayHelloAsyncFailure(),
      ])
    })
})

test('sayHelloAsync data error', () => {
  fetchMock.get(helloEndpointRoute(666), {})
  const store = mockStore()
  return store.dispatch(sayHelloAsync(666))
    .then(() => {
      expect(store.getActions()).toEqual([
        sayHelloAsyncRequest(),
        sayHelloAsyncFailure(),
      ])
    })
})
```

我們來看看這一段發生什麼事。首先我們藉由 `const mockStore = configureMockStore([thunkMiddleware])` 假裝 Redux store。如此一來，我們可以發送 actions 而不需要觸發 reducer。在每個測試中，我們藉由 `fetchMock.get()` 來假裝 `fetch`  然後讓它回傳任何我們想要的值。而我們實際上藉由 `expect()` 來測試一連串的由 store 發送出來的 actions。這多虧了 `store.getActions()` 這個來自於 `redux-mock-store`的函式。每個測試結尾我們用 `fetchMock.restore()` restore `fetch`。
Alright, Let's look at what's happening here. First we mock the Redux store using `const mockStore = configureMockStore([thunkMiddleware])`. By doing this we can dispatch actions without them triggering any reducer logic. For each test, we mock `fetch` using `fetchMock.get()` and make it return whatever we want. What we actually test using `expect()` is which series of actions have been dispatched by the store, thanks to the `store.getActions()` function from `redux-mock-store`. After each test we restore the normal behavior of `fetch` with `fetchMock.restore()`.

接下來測試比較簡單的 reducer 吧。
Let's now test our reducer, which is much easier.

- 建立 `src/client/reducer/hello.test.js` 內容包含：
- Create a `src/client/reducer/hello.test.js` file containing:

```js
import {
  sayHello,
  sayHelloAsyncRequest,
  sayHelloAsyncSuccess,
  sayHelloAsyncFailure,
} from '../action/hello'

import helloReducer from './hello'

let helloState

beforeEach(() => {
  helloState = helloReducer(undefined, {})
})

test('handle default', () => {
  expect(helloState.get('message')).toBe('Initial reducer message')
  expect(helloState.get('messageAsync')).toBe('Initial reducer message for async call')
})

test('handle SAY_HELLO', () => {
  helloState = helloReducer(helloState, sayHello('Test'))
  expect(helloState.get('message')).toBe('Test')
})

test('handle SAY_HELLO_ASYNC_REQUEST', () => {
  helloState = helloReducer(helloState, sayHelloAsyncRequest())
  expect(helloState.get('messageAsync')).toBe('Loading...')
})

test('handle SAY_HELLO_ASYNC_SUCCESS', () => {
  helloState = helloReducer(helloState, sayHelloAsyncSuccess('Test async'))
  expect(helloState.get('messageAsync')).toBe('Test async')
})

test('handle SAY_HELLO_ASYNC_FAILURE', () => {
  helloState = helloReducer(helloState, sayHelloAsyncFailure())
  expect(helloState.get('messageAsync')).toBe('No message received, please check your connection')
})
```

在每個測試之前，我們用 reducer 的預設值初始化 `helloState` (也就是 `switch` 陳述下 `default` 所執行的程式碼會回傳 `initialState`)。這些是非常明確的測試，我們根據收到何種 action 來驗證 reducer 會正確更新 `message` 或 `messageAsync`。
Before each test, we initialize `helloState` with the default result of our reducer (the `default` case of our `switch` statement in the reducer, which returns `initialState`). The tests are then very explicit, we just make sure the reducer updates `message` and `messageAsync` correctly depending on which action it received.

🏁 執行 `yarn test`。畫面應該會是一片綠。
🏁 Run `yarn test`. It should be all green.

下一章節：[06 - React Router, Server-Side Rendering, Helmet](06-react-router-ssr-helmet.md#readme)

回到 [上一章](04-webpack-react-hmr.md#readme)，或是回 [目錄](https://github.com/verekia/js-stack-from-scratch#table-of-contents)。
