+++
title = "React + font awesomeをwebpackで使う"
date = "2015-02-14"
categories= "front-end"
tags = ["React", "webpack", "fontawesome"]
+++

ライブラリ[react-fa](https://www.npmjs.com/package/react-fa)を使う。

Githubはこちら
https://github.com/andreypopp/react-fa

## インストール

パッケージをインストール

```
$ npm install react react-fa
```

その他必要なものをインストール

```
$ npm install jsx-loader style-loader css-loader url-loader -S
```

## webpackの設定

webpackのloaderにwebfontを読み込む設定を追加

```
loader:[
  {
    test: /\.woff2?(\?v=[0-9]\.[0-9]\.[0-9])?$/,
    loader: "url-loader?limit=10000&minetype=application/font-woff"
  },{
    test: /\.(ttf|eot|svg)(\?v=[0-9]\.[0-9]\.[0-9])?$/,
    loader: "file-loader"
  }
]
```

## 使い方

あとはjs,jsxファイルの中でrequireするだけ。

```
var Icon = require('react-fa');
```
### タグの使い方

使い方はこちらも見れば簡単に分かるはず。

http://andreypopp.github.io/react-fa/

`<Icon>`タグに`name`属性を付けその値をFont Awesomeのclass名から`fa-`を除いたものを指定

```
<Icon name="trash-o" />
```

回転アニメーションが必要なものは本家の場合、class名に`fa-spin`をつけるが、こちらでは`spin`属性をつける。

```
<Icon name="spinner" spin />
```

Rotatedも同様に`rotate`属性を付ける

```
<Icon name="shield" rotate="90" />
```


---
参考：
[Font Awesome Examples](http://fortawesome.github.io/Font-Awesome/examples/)
