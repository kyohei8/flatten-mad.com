---
title: 'ReactとStylusをwebpackで使うための開発環境構築'
date: '2014-12-22T04:35:56'
categories: 'front-end'
tags: ['React', 'Stylus', 'webpack']
---

巷で話題のReactを勉強するために、こちらも流行りそうなwebpackを使って環境構築してみた。
細かい内容はあまりわかっていないのでとりあえず手順だけメモ。

ソースは https://github.com/kyohei8/react-webpack-dev-env


## とりあえずのゴール

* webpackを使えるようにする(jsとstylusをcompileする)
* webpackを使ってReactをcompileできるようにする
* ローカルサーバをたててauto reload的なことをする


# Getting Started
## webpackのインストール

とりあえず

webpackの [installation](http://webpack.github.io/docs/installation.html) を参考にインストールする

```bash
$ npm install webpack -g
```

プロジェクトを作成して、ローカルにもwebpackをインストール

```
$ npm init
$ npm i webpack -S
```

## 各種Loaderをインストール
webpackではloaderと呼ばれる各種言語を変換するプラグインのようなものがある。  
今回利用するいくつかのものをインストールする

```
$ npm i jsx-loader style-loader stylus-loader css-loader -S
```

## webpack.config.jsの設定
webpackの設定ファイルを作成する

```
$ touch webpack.config.js
```

中身は以下の様な感じ

```js
// webpack.config.js
module.exports = {
  entry : './src/scripts/main.js',   // メインとなるのjsパス
  output: {
    path    : __dirname + '/dist/scripts/',
    filename: 'bundle.js'
  },
  module: {
    loaders: [{                     // require('**.js'); jsxのコンパイル設定
      test  : /\.js$/,
      loader: 'jsx-loader?harmony'  //es6の記法を使いたいのでharmonyを追加する
    }, {
      test  : /\.styl$/,            // require('**.styl')の設定
      loader: 'style!css!stylus'
    }]
  },
  externals: {
    // Reactをnpmからでなくグローバルから取得する
    // この設定がないとcompileが遅くなる
    'react': 'React'
  },
  resolve: {
    extensions: ['', '.js', '.json', '.coffee']
  }
};
```

## index.html を作成

`dist/index.html`を作成する
Reactとbundle.jsを読み込む。`div#example`にはReactでレンダリングされた部分が入る

Reactのファイルをhtml側で定義し、`webpack.config.js`の`externals`で外部変数として定義するのがポイント。これがないと毎回reactのライブラリもコンパイルされてしまうので、コンパイルが遅くなってしまう。


```html
<!DOCTYPE html>
<html>
<head lang="en">
<meta charset="UTF-8">
<title>React Demo</title>
<!-- Reactを読み込む -->
<script src="../node_modules/react/dist/react-with-addons.js"></script>
</head>
<body>
<div id="example"></div>
<script src="./scripts/bundle.js"></script>
</body>
</html>
```

## stylusファイルを作成

`src/styles/main.styl`を作成。中身は適当に

## main.jsを作成

`src/scripts/main.js`を作成。

```js
/*** @jsx React.DOM */
'use strict';

require('../styles/main.styl');  // stylを読み込む

var React = require('react');
var mount = document.getElementById('example');
// モジュールを読み込む
var Timer = require('./modules/timer');

var HelloMessage = React.createClass({
  // es6の書き方が使える
  render() {
    return <div><h1>Hello {this.props.name}</h1></div>;
  }
});

React.render(<HelloMessage name="my project" />, mount);
```

### 確認

`webpack`コマンドでコンパイルを実行。index.htmlを開いて動作しているかを確認。


```bash
$ webpack
Hash: da1d3a74b690478a3697
Version: webpack 1.4.13
Time: 700ms
    Asset  Size  Chunks             Chunk Names
bundle.js  9685       0  [emitted]  main
+ 6 hidden modules
```

# Reactのモジュールを追加する

`src/scripts/modules/timer.js`を追加

中身はhttp://facebook.github.io/react/index.html にあるTimerをとりあえずコピペしてes6にちょっとだけ変換

```js
/** @jsx React.DOM */
var React = require('react');

var Timer = React.createClass({
  getInitialState(){
    return {secondsElapsed: 0};
  },
  tick(){
    this.setState({secondsElapsed: this.state.secondsElapsed + 1});
  },
  componentDidMount(){
    this.interval = setInterval(this.tick, 1000);
  },
  componentWillUnmount(){
    clearInterval(this.interval);
  },
  render(){
    return (
      <div>Seconds Elapsed: {this.state.secondsElapsed}</div>
    );
  }
});
module.exports = Timer;
```
main.jsに追加

```js
/*** @jsx React.DOM */

...

// モジュールを読み込む
var Timer = require('./modules/timer');

  ...略...

  render() {
    return (
        <div>
          <h1>Hello {this.props.name}</h1>
          <Timer />
        </div>
    );
  }
  ...
```


### 確認

再度`webpack`コマンドでコンパイルを実行。index.htmlを開いて動作しているかを確認。


最終的にディレクトリ構成は以下の様な感じになった

```
.
├── node_modules
├── dist
│   ├── scripts
│   │   └── bundle.js  //生成されたjs
│   └── index.html
├── src
│   ├── scripts
│   │   ├── modules
│   │   │   └── timer.js //Reactモジュール
│   │   └── main.js      //Reactメイン
│   └── styles
│       └── main.styl    //StyleSheet
├── package.json
└── webpack.config.js
```


```
$ webpack
Hash: fa640905451e18801d94
Version: webpack 1.4.13
Time: 422ms
    Asset   Size  Chunks             Chunk Names
bundle.js  10401       0  [emitted]  main
+ 7 hidden modules
```
モジュールが1つ増えてますね。

これでJSXとStylus、JSXのモジュールがコンパイルできることを確認できました。

# 開発用ローカルサーバをたてる

ファイル更新時に、いちいちwebpackコマンドをうつのは面倒なので、
次にローカルサーバをたてて、自動リロードできるようにします。  
このへんはgruntやgulpでもwatchタスクを使えば同じようなことができますが、今回はnodeの普通のhttpサーバとwebpackの標準devサーバ([webpack-dev-server](http://webpack.github.io/docs/webpack-dev-server.html))を使います。


まずはインストール
```
$ npm i http-server -S
$ npm i webpack-dev-server -S
```

`http-server`は単純にhtmlをホストするだけで、実際のwatchとコンパイルは`webpack-dev-server`が行います。

index.htmlを以下のように変更

```html
<!DOCTYPE html>
<html>
<head lang="en">
<meta charset="UTF-8">
<title>React Demo</title>
<script src="../node_modules/react/dist/react-with-addons.js"></script>
</head>
<body>

<div id="example"></div>
<!--
変更時にリロードするため、webpack-dev-server scriptを読み込む。
webpack dev serverをport8090で動くので、そこに接続する。
-->
<script src="http://localhost:8090/webpack-dev-server.js"></script>
<!-- webpackによって生成された、bundle.jsを読み込むこのbundleはweppack-dev-serverから出力される-->
<script type="text/javascript" src="http://localhost:8090/assets/bundle.js"></script>
</body>
</html>
```

`webpack.config.js` のoutputにdev-server用の設定を追加

```js
...
output: {
  path    : __dirname + '/dist/scripts/',
  //webpack-dev-server用のアウトプットディレクトリ
  publicPath: 'http://localhost:8090/assets',
  filename: 'bundle.js'
  },
...
```
`package.json`のscriptを以下のようにする。

```json
"scripts": {
  "start": "npm run serve | npm run dev",
  "serve": "./node_modules/.bin/http-server -p 8080",
  "dev": "webpack-dev-server --progress --color --port 8090"
},
```

```
$ npm run start
```
で起動します。

`npm run serve`でローカルサーバをポート8080でたて、
`npm run dev`でwebpackのdevサーバをポート8090でたてている。

この状態で http://localhost:8080/dist/ にアクセス、bundle.jsの内容が表示されていることがわかる。
さらにmain.jsなどを変更すると自動でリロードがかかるようになる。

とりあえず、今日はここまで。
まだ、あんまり細かいところがわかっていないのでまとめたい。

---
参考リンク

* [petehunt/webpack-howto](https://github.com/petehunt/webpack-howto)
* [React with webpack - part 1 | On web and JavaScript](http://jslog.com/2014/10/02/react-with-webpack-part-1/)
