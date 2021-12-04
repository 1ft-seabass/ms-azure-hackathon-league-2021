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
