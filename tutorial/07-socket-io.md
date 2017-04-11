# 07 - Socket.IO

本章節的程式碼在[這裡](https://github.com/verekia/js-stack-walkthrough/tree/master/07-socket-io)。

> 💡 **[Socket.IO](https://github.com/socketio/socket.io)** 是個用來輕鬆操作 Websocket 的函式庫，提供方便的 API 以及作為不支援 Websocket 的瀏覽器的替代方案。

本章節中，我們將實作基本的客戶端與伺服器之間訊息交換範例。為了避免添加過多不重要的頁面與網頁元素，本章節不呈現任何UI，我們僅使用瀏覽器的 Console 視窗來展示功能。

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

These are the *type of messages* your client and your server will exchange. I suggest prefixing them with either `IO_CLIENT` or `IO_SERVER` to make it clearer *who* is sending the message. Otherwise, things can get pretty confusing when you have a lot of message types.

As you can see, we have a `IO_CLIENT_JOIN_ROOM`, because for the sake of demonstration, we are going to make clients join a room (like a chatroom). Rooms are useful to broadcast messages to specific groups of users.

- Create a `src/server/socket.js` file containing:

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

Okay, so in this file, we implement *how our server should react when clients connect and send messages to it*:

- When the client connects, we log it in the server console, and get access to the `socket` object, which we can use to communicate back with that client.
- When a client sends `IO_CLIENT_JOIN_ROOM`, we make it join the `room` it wants. Once it has joined a room, we send 3 demo messages: 1 message to every user, 1 message to users in that room, 1 message to that client only.
- When the client sends `IO_CLIENT_HELLO`, we log its message in the server console.
- When the client disconnects, we log it as well.

## Client-side

The client-side of things is going to look very similar.

- Edit `src/client/index.jsx` like so:

```js
// [...]
import setUpSocket from './socket'

// [at the very end of the file]
setUpSocket(store)
```

As you can see, we pass the Redux store to `setUpSocket`. This way whenever a Websocket message coming from the server should alter the client's Redux state, we can `dispatch` actions. We are not going to `dispatch` anything in this example though.

- Create a `src/client/socket.js` file containing:

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

What happens here should not be surprising if you understood well what we did on the server:

- As soon as the client is connected, we log it in the browser console and join the room `hello-1234` with a `IO_CLIENT_JOIN_ROOM` message.
- We then send `Hello!` with a `IO_CLIENT_HELLO` message.
- If the server sends us a `IO_SERVER_HELLO` message, we log it in the browser console.
- We also log any disconnection.

🏁 Run `yarn start` and `yarn dev:wds`, open `http://localhost:8000`. Then, open your browser console, and also look at the terminal of your Express server. You should see the Websocket communication between your client and server.

Next section: [08 - Bootstrap, JSS](08-bootstrap-jss.md#readme)

Back to the [previous section](06-react-router-ssr-helmet.md#readme) or the [table of contents](https://github.com/verekia/js-stack-from-scratch#table-of-contents).
