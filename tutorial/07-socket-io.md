# 07 - Socket.IO

這一節的程式碼在[這裡](https://github.com/verekia/js-stack-walkthrough/tree/master/07-socket-io)。

> 💡 **[Socket.IO](https://github.com/socketio/socket.io)** 是個用來輕鬆操作 Websocket 的函式庫，提供方便的 API 以及作為不支援 Websocket 的瀏覽器的替代方案。

這一節中，我們將實作基本的客戶端與伺服器之間訊息交換範例。為了避免添加過多不重要的頁面與網頁元素，本章節不呈現任何UI，我們僅使用瀏覽器的 Console 視窗來展示功能。

- 在終端機執行 `yarn add socket.io socket.io-client`

## 伺服器端

- 將 `src/server/index.js` 改成以下內容:

```js
// @flow

import compression from 'compression'
import express from 'express'
import { Server } from 'http'
import socketIO from 'socket.io'

import routing from './routing'
import { WEB_PORT, STATIC_PATH } from '../shared/config'
import { isProd } from '../shared/util'
import setUpSocket from './socket'

const app = express()
// flow-disable-next-line
const http = Server(app)
const io = socketIO(http)
setUpSocket(io)

app.use(compression())
app.use(STATIC_PATH, express.static('dist'))
app.use(STATIC_PATH, express.static('public'))

routing(app)

http.listen(WEB_PORT, () => {
  // eslint-disable-next-line no-console
  console.log(`Server running on port ${WEB_PORT} ${isProd ? '(production)' :
    '(development).\nKeep "yarn dev:wds" running in an other terminal'}.`)
})
```

特別注意如果要讓 Socket.IO 正常運作，你需要用 `http` 的 `Server` 來聽 (`listen`) 傳進來的 Web 請求，而不是 Express 的 `app`。不過這其實不會太麻煩，因為所有跟 Websocket 有關的細節都寫在另一個叫 `setUpSocket` 的檔案裡。

- 在檔案 `src/shared/config.js` 裡加入下列常數：

```js
export const IO_CONNECT = 'connect'
export const IO_DISCONNECT = 'disconnect'
export const IO_CLIENT_HELLO = 'IO_CLIENT_HELLO'
export const IO_CLIENT_JOIN_ROOM = 'IO_CLIENT_JOIN_ROOM'
export const IO_SERVER_HELLO = 'IO_SERVER_HELLO'
```

這些是你的伺服器與客戶端會交換的*訊息類別*。我建議在各類別前面加上 `IO_CLIENT` 或 `IO_SERVER` 以區別訊息由*誰*送出，否則訊息類別變多時很容易搞不清楚。

現在我們有個類別叫作 `IO_CLIENT_JOIN_ROOM` 是因為在展示成果時，我們會讓所有客戶端加入一個像聊天室一樣的房間 (Room)。這種房間很適合用來將訊息傳給特定使用者群組。

- 建立檔案 `src/server/socket.js` 並貼上以下內容:

```js
// @flow

import {
  IO_CONNECT,
  IO_DISCONNECT,
  IO_CLIENT_JOIN_ROOM,
  IO_CLIENT_HELLO,
  IO_SERVER_HELLO,
} from '../shared/config'

/* eslint-disable no-console */
const setUpSocket = (io: Object) => {
  io.on(IO_CONNECT, (socket) => {
    console.log('[socket.io] A client connected.')

    socket.on(IO_CLIENT_JOIN_ROOM, (room) => {
      socket.join(room)
      console.log(`[socket.io] A client joined room ${room}.`)

      io.emit(IO_SERVER_HELLO, 'Hello everyone!')
      io.to(room).emit(IO_SERVER_HELLO, `Hello clients of room ${room}!`)
      socket.emit(IO_SERVER_HELLO, 'Hello you!')
    })

    socket.on(IO_CLIENT_HELLO, (clientMessage) => {
      console.log(`[socket.io] Client: ${clientMessage}`)
    })

    socket.on(IO_DISCONNECT, () => {
      console.log('[socket.io] A client disconnected.')
    })
  })
}
/* eslint-enable no-console */

export default setUpSocket
```

現在我們在這個檔案裡實作了*客戶端連上伺服器時該有什麼動作，以及如何傳訊息給客戶端*。

- 客戶端連上伺服器時，我們在伺服器的 Console 輸出相關通知，並拿到 `socket` 這個物件。我們將用 `socket` 來和對應的客戶端溝通。
- 當客戶端送一個 `IO_CLIENT_JOIN_ROOM` 訊息過來時，我們把他加入他想去的那個 `room`。進入房間後，我們同時送出 3 種展示用的訊息：給所有使用者的、給那個房間的使用者以及單獨給那個新使用者的。
- 當客戶端送一個 `IO_CLIENT_HELLO` 訊息過來時，我們在伺服器的 Console 輸出他送來的訊息。
- 當客戶端離線時，我們也在伺服器 Console 輸出這件事。

## 客戶端

客戶端的程式碼看起來其實也差不多。

- 在檔案 `src/client/index.jsx` 中加入這兩行:

```js
// [...]
import setUpSocket from './socket'

// [檔案的最下面]
setUpSocket(store)
```

現在我們把 Redux store 傳給 `setUpSocket`，如此一來無論從伺服器傳過來的 Websocket 訊息什麼時候會改變客戶端 Redux 的狀態，我們都可以 `dispatch` 這些動作，不過在這個範例裡我們不會 `dispatch` 任何東西。

- 建立檔案 `src/client/socket.js` 並貼上以下內容:

```js
// @flow

import socketIOClient from 'socket.io-client'

import {
  IO_CONNECT,
  IO_DISCONNECT,
  IO_CLIENT_HELLO,
  IO_CLIENT_JOIN_ROOM,
  IO_SERVER_HELLO,
  } from '../shared/config'

const socket = socketIOClient(window.location.host)

/* eslint-disable no-console */
// eslint-disable-next-line no-unused-vars
const setUpSocket = (store: Object) => {
  socket.on(IO_CONNECT, () => {
    console.log('[socket.io] Connected.')
    socket.emit(IO_CLIENT_JOIN_ROOM, 'hello-1234')
    socket.emit(IO_CLIENT_HELLO, 'Hello!')
  })

  socket.on(IO_SERVER_HELLO, (serverMessage) => {
    console.log(`[socket.io] Server: ${serverMessage}`)
  })

  socket.on(IO_DISCONNECT, () => {
    console.log('[socket.io] Disconnected.')
  })
}
/* eslint-enable no-console */

export default setUpSocket
```

如果你清楚了解我們在伺服器上做的事情，這些結果看起來就再正常不過了。

- 客戶端一連上伺服器，登入訊息會顯示在瀏覽器的 Console 視窗裡，並送出 `IO_CLIENT_JOIN_ROOM` 訊息以加入編號 `hello-1234` 的房間。
- 然後用 `IO_CLIENT_HELLO` 訊息送出 `Hello!` 到伺服器。
- 如果伺服器回送 `IO_SERVER_HELLO` 訊息給我們，也會顯示在瀏覽器的 Console 視窗裡。
- 離線時也會顯示相關訊息。

🏁 在終端機執行 `yarn start` 和 `yarn dev:wds`， 在瀏覽器中連到 `http://localhost:8000`，然後打開瀏覽器的 Console 視窗，同時注意 Express 伺服器的終端機視窗，應該可以看到你的客戶端與伺服器透過 Websocket 的溝通情形。

下一節: [08 - Bootstrap, JSS](08-bootstrap-jss.md#readme)

回到 [上一節](06-react-router-ssr-helmet.md#readme) 或 [內容目錄](https://github.com/verekia/js-stack-from-scratch#內容目錄).
