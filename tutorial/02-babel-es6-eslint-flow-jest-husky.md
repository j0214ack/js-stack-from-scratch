# 02 - Babel, ES6, ESLint, Flow, Jest, 與 Husky

本章的程式碼可以在 [這裡](https://github.com/verekia/js-stack-walkthrough/tree/master/02-babel-es6-eslint-flow-jest-husky) 取得。

我們將會使用一些 ES6 語法，它比起「老舊」的 ES5 語法進步了許多。但是所有的瀏覽器與 JS 環境都能很好地支援 ES5 ，而非 ES6 。這就是我們需要使用 Babel 工具的時機了。

## Babel

> 💡 **[Babel](https://babeljs.io/)** 是一個將 ES6 程式碼轉譯成 ES5 的編譯器。它非常的模組化，而且可以在各種不同的[環境](https://babeljs.io/docs/setup/)中運作。目前在 React 社群中，Babel 是最被接受的 ES5 編譯器。

- 將 `index.js` 移動到一個新的 `src` 資料夾。你會在這個資料夾裡撰寫你的 ES6 程式碼。將與 `color` 有關的程式碼從 `index.js` 中移除，並且將內容換成：

```js
const str = 'ES6'
console.log(`Hello ${str}`)
```

我們在這裡使用了 *template string* 。這是 ES6 的特色，讓我們可以在字串裡面透過 `${}` 植入變數，而不用經由字串串接。請注意， *template string* 使用的是 **反引號**。

- 執行 `yarn add --dev babel-cli` 安裝 Babel 的 CLI。
（譯注： CLI 是 command-line interface 或 command language interpreter ， 也就是在終端機的介面。）

Babel CLI 共有[兩個可執行指令](https://babeljs.io/docs/usage/cli/)：
1. `babel` ， `babel` 將 ES6 檔案編譯成新的 ES5 檔案。
2. `babel-node` ， `babel-node` 可以取代 `node` 執行檔，讓你直接執行 ES6 檔案。 `babel-node` 很非常適合在開發環境中使用，但對正式環境來說太過笨重。
在本章節我們將會使用 `babel-node` 設置開發環境，而在下一個章節我們會使用 `babel` 來製作 ES5 檔案到正式環境。

- 在 `package.json` 中的 `start` 腳本，將 `node .` 替換成 `babel-node src` 。(`index.js` 是 Node 預設查看的檔案，所以我們可以將 `index.js` 省略。）

如果你現在試著嘗試執行 `yarn start`，應該能夠印出正確的結果。但 Babel 實際上沒有做任何事，因為我們沒有給它想要套用哪些轉換的資訊。他能夠印出正確結果的唯一理由是因為 Node 不用透過 Babel 就能夠理解 ES6。在某些瀏覽器上或比較舊的 Node 版本裡，可能就不會這麼順利了！

- 執行 `yarn add --dev babel-preset-env` 來安裝 Babel 預設套件 `env` ，這個套件包含了最近期的 Babel 支援的 ECMAScript 語法設定。

- 在你的專案的根目錄中創造 `.babelrc` 檔案，這是一個 JSON 格式的檔案，作為你的 Babel 設置檔。在當中撰寫以下內容，好讓 Babel 使用 `env` 預設：


```json
{
  "presets": [
    "env"
  ]
}
```

🏁 `yarn start` 應該依舊能夠運作。但現在 Babel 真的有在做點事情了。但我們不太能確定是不是真的，因為我們正在使用 `babel-node` 來運行 ES6 。但很快地，當到本章的 [ES6 模組語法](#ES6 模組語法) 段落時，我們就能看到ES6 程式碼有被改變的證據。

## ES6

> 💡 **[ES6](http://es6-features.org/)**: 是 JavaScript 語言最顯著的改進。 ES6 的特色多到無法一一列舉，但典型的 ES6 程式碼會：使用 `class` 來創造類別、 `const` 、 `let` 、 template string 、以及箭頭函式 (`(text) => { console.log(text) }`)。

### 創造一個 ES6 類別

- 創造一個新檔案叫做 `src/dog.js` ，裡面包含下列類別：

```js
class Dog {
  constructor(name) {
    this.name = name
  }

  bark() {
    return `Wah wah, I am ${this.name}`
  }
}

module.exports = Dog
```

如果你曾經在其他語言用過物件導向，這個語法應該不會看起來太意外。但對 JavaScript 來說，這是一個相對較新的語法。類別透過 `module.exports` 向外公開。

在 `src/index.js` 裡，撰寫以下程式碼：

```js
const Dog = require('./dog')

const toby = new Dog('Toby')

console.log(toby.bark())
```

如你所見，不像我們之前用的社群撰寫的 `color` 套件，當使用我們自己的檔案時，會在 `require()` 裡使用 `./`


🏁  執行 `yarn start` ，應該會印出：「 Wah wah, I am Toby 」。


### ES6 模組語法

將 `const Dog = require('./dog')` 更換成 `import Dog from './dog'`。
`import` 是新的 ES6 模組語法。（ `require()` 則是 「 CommonJS」 模組的語法。）這個語法沒有被 NodeJS 直接支援，所以這將會是 Babel 有正確處理 ES6 檔案的證據。

同樣地，在 `dog.js` 裡，將 `module.exports = Dog` 換成 `export default Dog` 。

🏁 `yarn start` 應該依舊會顯示「 Wah wah, I am Toby 」。

## ESLint

> 💡 **[ESLint](http://eslint.org)** 用來處理 ES6 的 linter。 linter 會給你程式碼格式的建議，強迫你和你的團隊在程式碼當中有一致的風格。同時也是一個學習 JavaScript 的好方法，因為你寫錯文法時， ESlint 都會抓到並且提醒你。

ESLint 透過 *rules* 運作，ESlint 有[許多規則](http://eslint.org/docs/rules/)，我們會使用 Airbnb 做出來的設置 (config)，而不會自己一條一條去設定。 Airbnb 的設置會用到不少插件(plugins)，我們需要將這些插件安裝好。


Check out Airbnb's most recent [instructions](https://www.npmjs.com/package/eslint-config-airbnb) to install the config package and all its dependencies correctly. As of 2017-02-03, they recommend using the following command in your terminal:

```sh
npm info eslint-config-airbnb@latest peerDependencies --json | command sed 's/[\{\},]//g ; s/: /@/g' | xargs yarn add --dev eslint-config-airbnb@latest
```

It should install everything you need and add `eslint-config-airbnb`, `eslint-plugin-import`, `eslint-plugin-jsx-a11y`, and `eslint-plugin-react` to your `package.json` file automatically.

**Note**: I've replaced `npm install` by `yarn add` in this command. Also, this won't work on Windows, so take a look at the `package.json` file of this repository and just install all the ESLint-related dependencies manually using `yarn add --dev packagename@^#.#.#` with `#.#.#` being the versions given in `package.json` for each package.

- Create an `.eslintrc.json` file at the root of your project, just like we did for Babel, and write the following to it:

```json
{
  "extends": "airbnb"
}
```

We'll create an NPM/Yarn script to run ESLint. Let's install the `eslint` package to be able to use the `eslint` CLI:

- Run `yarn add --dev eslint`

Update the `scripts` of your `package.json` to include a new `test` task:

```json
"scripts": {
  "start": "babel-node src",
  "test": "eslint src"
},
```

Here we just tell ESLint that want to lint all JavaScript files under the `src` folder.

We will use this standard `test` task to run a chain of all the commands that validate our code, whether it's linting, type checking, or unit testing.

- Run `yarn test`, and you should see a whole bunch of errors for missing semicolons, and a warning for using `console.log()` in `index.js`. Add `/* eslint-disable no-console */` at the top of our `index.js` file to allow the use of `console` in this file.

**Note**: If you're on Windows, make sure you configure your editor and Git to use Unix LF line endings and not Windows CRLF. If your project is only used in Windows environments, you can add `"linebreak-style": [2, "windows"]` in ESLint's `rules` array (see the example below) to enforce CRLF instead.

### Semicolons

Alright, this is probably the most heated debate in the JavaScript community, let's talk about it for a minute. JavaScript has this thing called Automatic Semicolon Insertion, which allows you to write your code with or without semicolons. It really comes down to personal preference and there is no right and wrong on this topic. If you like the syntax of Python, Ruby, or Scala, you will probably enjoy omitting semicolons. If you prefer the syntax of Java, C#, or PHP, you will probably prefer using semicolons.

Most people write JavaScript with semicolons, out of habit. That was my case until I tried going semicolon-less after seeing code samples from the Redux documentation. At first it felt a bit weird, simply because I was not used to it. After just one day of writing code this way I could not see myself going back to using semicolons at all. They felt so cumbersome and unnecessary. A semicolon-less code is easier on the eyes in my opinion, and is faster to type.

I recommend reading the [ESLint documentation about semicolons](http://eslint.org/docs/rules/semi). As mentioned in this page, if you're going semicolon-less, there are some rather rare cases where semicolons are required. ESLint can protect you from such cases with the `no-unexpected-multiline` rule. Let's set up ESLint to safely go semicolon-less in `.eslintrc.json`:

```json
{
  "extends": "airbnb",
  "rules": {
    "semi": [2, "never"],
    "no-unexpected-multiline": 2
  }
}
```

🏁 Run `yarn test`, and it should now pass successfully. Try adding an unnecessary semicolon somewhere to make sure the rule is set up correctly.

I am aware that some of you will want to keep using semicolons, which will make the code provided in this tutorial inconvenient. If you are using this tutorial just for learning, I'm sure it will remain bearable to learn without semicolons, until going back to using them on your real projects. If you want to use the code provided in this tutorial as a boilerplate though, it will require a bit of rewriting, which should be pretty quick with ESLint set to enforce semicolons to guide you through the process. I apologize if you're in such case.

### ESLint in your editor

This chapter set you up with ESLint in the terminal, which is great for catching errors at build time / before pushing, but you also probably want it integrated to your IDE for immediate feedback. Do NOT use your IDE's native ES6 linting. Configure it so the binary it uses for linting is the one in your `node_modules` folder instead. This way it can use all of your project's config, the Airbnb preset, etc. Otherwise you will just get some generic ES6 linting.

## Flow

> 💡 **[Flow](https://flowtype.org/)**: A static type checker by Facebook. It detects inconsistent types in your code. For instance, it will give you an error if you try to use a string where should be using a number.

Right now, our JavaScript code is valid ES6 code. Flow can analyze plain JavaScript to give us some insights, but in order to use its full power, we need to add type annotations in our code, which will make it non-standard. We need to teach Babel and ESLint what those type annotations are in order for these tools to not freak out when parsing our files.

- Run `yarn add --dev flow-bin babel-preset-flow babel-eslint eslint-plugin-flowtype`

`flow-bin` is the binary to run Flow in our `scripts` tasks, `babel-preset-flow` is the preset for Babel to understand Flow annotations, `babel-eslint` is a package to enable ESLint *to rely on Babel's parser* instead of its own, and `eslint-plugin-flowtype` is an ESLint plugin to lint Flow annotations. Phew.

- Update your `.babelrc` file like so:

```json
{
  "presets": [
    "env",
    "flow"
  ]
}
```

- And update `.eslintrc.json` as well:

```json
{
  "extends": [
    "airbnb",
    "plugin:flowtype/recommended"
  ],
  "plugins": [
    "flowtype"
  ],
  "rules": {
    "semi": [2, "never"],
    "no-unexpected-multiline": 2
  }
}
```

**Note**: The `plugin:flowtype/recommended` contains the instruction for ESLint to use Babel's parser. If you want to be more explicit, feel free to add `"parser": "babel-eslint"` in `.eslintrc.json`.

I know this is a lot to take in, so take a minute to think about it. I'm still amazed that it is even possible for ESLint to use Babel's parser to understand Flow annotations. These 2 tools are really incredible for being so modular.

- Chain `flow` to your `test` task:

```json
"scripts": {
  "start": "babel-node src",
  "test": "eslint src && flow"
},
```

- Create a `.flowconfig` file at the root of your project containing:

```flowconfig
[options]
suppress_comment= \\(.\\|\n\\)*\\flow-disable-next-line
```

This is a little utility that we set up to make Flow ignore any warning detected on the next line. You would use it like this, similarly to `eslint-disable`:

```js
// flow-disable-next-line
something.flow(doesnt.like).for.instance()
```

Alright, we should be all set for the configuration part.

- Add Flow annotations to `src/dog.js` like so:

```js
// @flow

class Dog {
  name: string

  constructor(name: string) {
    this.name = name
  }

  bark() {
    return `Wah wah, I am ${this.name}`
  }
}

export default Dog
```

The `// @flow` comment tells Flow that we want this file to be type-checked. For the rest, Flow annotations are typically a colon after a function parameter or a function name. Check out the [documentation](https://flowtype.org/docs/quick-reference.html) for more details.

- Add `// @flow` at the top of `index.js` as well.

`yarn test` should now both lint and type-check your code fine.

There are 2 things that I want you to try:

- In `dog.js`, replace `constructor(name: string)` by `constructor(name: number)`, and run `yarn test`. You should get a **Flow** error telling you that those types are incompatible. That means Flow is set up correctly.

- Now replace `constructor(name: string)` by `constructor(name:string)`, and run `yarn test`. You should get an **ESLint** error telling you that Flow annotations should have a space after the colon. That means the Flow plugin for ESLint is set up correctly.

🏁 If you got the 2 different errors working, you are all set with Flow and ESLint! Remember to put the missing space back in the Flow annotation.

### Flow in your editor

Just like with ESLint, you should spend some time configuring your editor / IDE to give you immediate feedback when Flow detects issues in your code.

## Jest

> 💡 **[Jest](https://facebook.github.io/jest/)**: A JavaScript testing library by Facebook. It is very simple to set up and provides everything you would need from a testing library right out of the box. It can also test React components.

- Run `yarn add --dev jest babel-jest` to install Jest and the package to make it use Babel.

- Add the following to your `.eslintrc.json` at the root of the object to allow the use of Jest's functions without having to import them in every test file:

```json
"env": {
  "jest": true
}
```

- Create a `src/dog.test.js` file containing:

```js
import Dog from './dog'

test('Dog.bark', () => {
  const testDog = new Dog('Test')
  expect(testDog.bark()).toBe('Wah wah, I am Test')
})
```

- Add `jest` to your `test` script:

```json
"scripts": {
  "start": "babel-node src",
  "test": "eslint src && flow && jest --coverage"
},
```

The `--coverage` flag makes Jest generate coverage data for your tests automatically. This is useful to see which parts of your codebase lack testing. It writes this data into a `coverage` folder.

- Add `/coverage/` to your `.gitignore`

🏁 Run `yarn test`. After linting and type checking, it should run Jest tests and show a coverage table. Everything should be green!

## Git Hooks with Husky

> 💡 **[Git Hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)**: Scripts that are run when certain actions like a commit or a push occur.

Okay so we now have this neat `test` task that tells us if our code looks good or not. We're going to set up Git Hooks to automatically run this task before every `git commit` and `git push`, which will prevent us from pushing bad code to the repository if it doesn't pass the `test` task.

[Husky](https://github.com/typicode/husky) is a package that makes this very easy to set up Git Hooks.

- Run `yarn add --dev husky`

All we have to do is to create two new tasks in `scripts`, `precommit` and `prepush`:

```json
"scripts": {
  "start": "babel-node src",
  "test": "eslint src && flow && jest --coverage",
  "precommit": "yarn test",
  "prepush": "yarn test"
},
```

🏁 If you now try to commit or push your code, it should automatically run the `test` task.

**Note**: If you are pushing right after a commit, you can use `git push --no-verify` to avoid running all the tests again.

Next section: [03 - Express, Nodemon, PM2](03-express-nodemon-pm2.md#readme)

Back to the [previous section](01-node-yarn-package-json.md#readme) or the [table of contents](https://github.com/verekia/js-stack-from-scratch#table-of-contents).
