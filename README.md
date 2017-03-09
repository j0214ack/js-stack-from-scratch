# JavaScript Stack 從零開始

[![Build Status](https://travis-ci.org/verekia/js-stack-from-scratch.svg?branch=master)](https://travis-ci.org/verekia/js-stack-from-scratch)
[![Release](https://img.shields.io/github/release/verekia/js-stack-from-scratch.svg?style=flat-square)](https://github.com/verekia/js-stack-from-scratch/releases)
[![Dependencies](https://img.shields.io/david/verekia/js-stack-boilerplate.svg?style=flat-square)](https://david-dm.org/verekia/js-stack-boilerplate)
[![Dev Dependencies](https://img.shields.io/david/dev/verekia/js-stack-boilerplate.svg?style=flat-square)](https://david-dm.org/verekia/js-stack-boilerplate?type=dev)
[![Gitter](https://img.shields.io/gitter/room/js-stack-from-scratch/Lobby.svg?style=flat-square)](https://gitter.im/js-stack-from-scratch/)

[![React](/img/react-padded-90.png)](https://facebook.github.io/react/)
[![Redux](/img/redux-padded-90.png)](http://redux.js.org/)
[![React Router](/img/react-router-padded-90.png)](https://github.com/ReactTraining/react-router)
[![Flow](/img/flow-padded-90.png)](https://flowtype.org/)
[![ESLint](/img/eslint-padded-90.png)](http://eslint.org/)
[![Jest](/img/jest-padded-90.png)](https://facebook.github.io/jest/)
[![Yarn](/img/yarn-padded-90.png)](https://yarnpkg.com/)
[![Webpack](/img/webpack-padded-90.png)](https://webpack.github.io/)
[![Bootstrap](/img/bootstrap-padded-90.png)](http://getbootstrap.com/)

歡迎來到現代 JavaScript Stack 教學: **JavaScript Stack 從零開始**。

(譯註：此為繁體中文翻譯，我會盡力更新，可到原作者的 [repo](https://github.com/verekia/js-stack-from-scratch) 查看最新版本，若您希望加入一起翻譯中文，或者有找到錯字或用語修正等等，可以 fork 此 repo 並且發送 pull request 給我。若想要翻譯整個章節可以到這個 [issue](https://github.com/j0214ack/js-stack-from-scratch/issues) 留言認領，避免大家重複翻譯。)

> 🎉 **這是此教學的第二版，相較 2016 年的釋出，有不少重大更新，看看[更改日誌](/CHANGELOG.md)!**

這是一個簡單直接的組裝 JavaScript Stack 引導。讀者需要一些程式知識，以及一些 JavaScript 基礎。 **本引導的重點在將各種工具結合在一起**，並對每個工具給您 **最簡易的範例**。你可以把這個教學看成是 *從零開始撰寫模板的一個方式*。由於本教的目標是將各種工具組裝在一起，我不會進入各個工具的運作細節。如果您希望有對不同工具有更深入的理解，請參考各工具的文件，或找到他們的教學。

如果你只是要用少數的工具建立綺一個簡單的網頁，你不需要使用整個 Stack（像是 Browserify/Webpack + Babel + jQuery 就足夠讓你在不同的檔案裡撰寫 ES6 語法的程式碼），但如果你希望建造一個可以擴展規模的網頁應用程式，或需要協助將一切設定好，本教學非常適合您。


本教學中有一大部分會使用到 React 。如果你是初學者，並且只是想要學習 React，或者有人剛加入使用 React 的團隊並且需要一個學習的空間，我推薦用  [create-react-app](https://github.com/facebookincubator/create-react-app) 以預設好的設定迅速幫助您建立起 React 環境。在這個教學裡你不會使用預設設定，因為我希望您了解所有在檯面下發生的事情。


每章節都有程式碼範例，你可以透過 `yarn && yarn start` 執行這些範例。但我會建議您跟著 **一步一步的指引** 自己從零開始撰寫。


最終的程式碼可以在 [JS-Stack-Boilerplate repository](https://github.com/verekia/js-stack-boilerplate) 和 [releases](https://github.com/verekia/js-stack-from-scratch/releases) 取得。 也可以參考 [live demo](https://js-stack.herokuapp.com/)。


本教學可以在 Linux, macOS, and Windows 運作。


## 內容目錄

[01 - Node, Yarn, `package.json`](/tutorial/01-node-yarn-package-json.md#readme)

[02 - Babel, ES6, ESLint, Flow, Jest, Husky](/tutorial/02-babel-es6-eslint-flow-jest-husky.md#readme)

[03 - Express, Nodemon, PM2](/tutorial/03-express-nodemon-pm2.md#readme)

[04 - Webpack, React, HMR](/tutorial/04-webpack-react-hmr.md#readme)

[05 - Redux, Immutable, Fetch](/tutorial/05-redux-immutable-fetch.md#readme)

[06 - React Router, Server-Side Rendering, Helmet](/tutorial/06-react-router-ssr-helmet.md#readme)

[07 - Socket.IO](/tutorial/07-socket-io.md#readme)

[08 - Bootstrap, JSS](/tutorial/08-bootstrap-jss.md#readme)

[09 - Travis, Coveralls, Heroku](/tutorial/09-travis-coveralls-heroku.md#readme)

## 接下來的內容

設置你的編輯器（首先是 Atom）、MongoDB、Progressive Web APP。

## 翻譯

如果您希望增加您的翻譯，請從閱讀[翻譯建議](/how-to-translate.md) 開始

### V2

馬上就要有你的連結啦！ ;)

查看 [正在進行中的翻譯](https://github.com/verekia/js-stack-from-scratch/issues/147).

### V1

- [中文](https://github.com/pd4d10/js-stack-from-scratch) by [@pd4d10](http://github.com/pd4d10)
- [Italiano](https://github.com/fbertone/js-stack-from-scratch) by [Fabrizio Bertone](https://github.com/fbertone)
- [日本語](https://github.com/takahashim/js-stack-from-scratch) by [@takahashim](https://github.com/takahashim)
- [Русский](https://github.com/UsulPro/js-stack-from-scratch) by [React Theming](https://github.com/sm-react/react-theming)
- [ไทย](https://github.com/MicroBenz/js-stack-from-scratch) by [MicroBenz](https://github.com/MicroBenz)

## 致謝

Created by [@verekia](https://twitter.com/verekia) – [verekia.com](http://verekia.com/).

License: MIT
