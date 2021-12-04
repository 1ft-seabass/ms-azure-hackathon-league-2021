# Azure Functions と Line Bot

## 概要

この ページは Azure Functions と Line Bot を連携してみたナレッジを残しています。

## その前に Azure Functions の使い方を知りたい方は

分かりやすい教材がたくさんある Azure Learn で 私の方が Azure Functions 関連の教材をまとめてみたので参考にしてください。

[ハッカソンリーグ2021 ルーキーリーグ 田中正吾の Azure Functions コレクション](https://docs.microsoft.com/ja-jp/users/tseigo/collections/zkmrf47me0nozg)

## ベースにした記事

いろいろな記事がありますが、以下の記事が内容としてちょうどよかったので、こちらをベースに進めてみました。

[Azure FunctionsでLINE Bot作成 \- Qiita](https://qiita.com/RyogaTakao/items/a86522d560178f83652c)

こちらをすすめると、このように、メッセージを投稿するとオウム返しする LINE Bot ができます。

![image](https://i.gyazo.com/396db783d1d798592a5da24bf2372bd7.jpg)

ソースコードは記事内のものですが、こちらです。アクセストークン `<YOUR_TOKEN>` とチャンネルシークレット `<YOUR_SECRET>` を、自分のお使いになる実際の LINE Bot の設定に書き換えましょう。

```js
const line = require('@line/bot-sdk');

const config = {
    channelAccessToken: '<YOUR_TOKEN>',
    channelSecret:  '<YOUR_SECRET>',
};

const client = new line.Client(config);

module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    if (req.query.message || (req.body && req.body.events)) {
        if (req.body && req.body.events[0]) {
            message = {
                type: "text",
                text: req.body.events[0].message.text
            }
            console.log(message);
            if (req.body.events[0].replyToken) {
                client.replyMessage(req.body.events[0].replyToken, message);
            }
        }
        else {
            context.res = {
                status: 200,
                body: req.query.message
            };
        }
    }
    else {
        context.res = {
            status: 200,
            body: "Please check the query string in the request body"
        };
    };
};
```

## 柴犬 API とつなげる

### axios ライブラリをインストール

HTTP の API にアクセスしやすい axios ライブラリを準備しましょう。

- axios
    - 公式ドキュメント [axios/axios: Promise based HTTP client for the browser and node\.js](https://github.com/axios/axios)

![image](https://i.gyazo.com/3079b775e0b44fc2dfe2c5200555daee.png)

今回使う Functions の左メニューから 開発ツール > コンソールを開きます。

![image](https://i.gyazo.com/965e839c968ed94e85bde01fc2d84113.png)

開きました。

```sh
npm i axios
```

で axios をインストールします。

![image](https://i.gyazo.com/d482c0dc3f8c5fa91c139286754071c2.png)

しばらくくるくる回るので待ちます。

![image](https://i.gyazo.com/a13d67092da584cdfd543f7805659d08.png)

インストールしました。この赤文字は気にしなくて大丈夫です。

![image](https://i.gyazo.com/6957728a4aeada9c0c4f871e0785dfbf.png)

`npm list axios` と入力して Enter キーを入力すると、インストールされたか確認できます。

### 柴犬画像取得の関数

オウム返しでうまくいった関数をこちらに書き換えましょう。また、アクセストークン `<YOUR_TOKEN>` とチャンネルシークレット `<YOUR_SECRET>` を、自分のお使いになる実際の LINE Bot の設定に書き換えましょう。

```js
const line = require('@line/bot-sdk');
const axios = require('axios');

const config = {
    channelAccessToken: '<YOUR_TOKEN>',
    channelSecret:  '<YOUR_SECRET>',
};

const client = new line.Client(config);

module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');
    
    if (req.query.message || (req.body && req.body.events)) {
        if (req.body && req.body.events[0]) {
            
            if( req.body.events[0].message.text.indexOf('柴犬') != -1 ){  // 柴犬の文字が含まれているとき
                // 柴犬 API にアクセス
                const responseShibaAPI = await axios.get('http://shibe.online/api/shibes?count=1&urls=true&httpsUrls=true');
                context.log(responseShibaAPI.data);
                // ランダムで帰ってきた柴犬画像を取得
                const imageURL = responseShibaAPI.data[0];
                // 画像表示
                message = {
                    type: "image",
                    originalContentUrl: imageURL,
                    previewImageUrl: imageURL
                }
            } else {
                // オウム返し
                message = {
                    type: "text",
                    text: req.body.events[0].message.text
                }
            }

            console.log(message);

            if (req.body.events[0].replyToken) {
                client.replyMessage(req.body.events[0].replyToken, message);
            }
        }
        else {
            context.res = {
                status: 200,
                body: req.query.message
            };
        }
    }
    else {
        context.res = {
            status: 200,
            body: "Please check the query string in the request body"
        };
    };
};
```

### 実行してみる

柴犬がメッセージに含まれていると柴犬画像が返答され。それ以外は、今まで通り、オウム返しします。

![image](https://i.gyazo.com/425bc378dc03814d66a88548e255b36c.jpg)
