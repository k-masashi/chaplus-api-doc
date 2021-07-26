# Chaplus API β

## バージョン履歴

### v.0.60 (2021.07.26)
[日本語用GPT-2](https://github.com/rinnakk/japanese-gpt2)
をファインチューニングしたモデルを利用して一部の応答を返すことができるようになりました。(試験導入) \
※ GPT-2を用いた応答はScore=0.7が決め打ちで返されます。 \
※ GPT-2を用いた応答のみ利用したい場合は、リクエストで「useGpt2」を指定します。 \


### v.0.51 (2020.12.20)

発話をカスタマイズする際に個別の発話候補を設定できるようになりました。 \
(before)
```json
"utterancePairs":[
    {
        "utterance":"おはよう！",
        "response":"今日も1日頑張って！",
    },
    {
        "utterance":"こんにちは",
        "response":"ハロー"
    }
]
```

(after)
```json
"utterancePairs":[
    {
        "utterance":"おはよう！",
        "response":"今日も1日頑張って！",
        "options":"顔洗ってスッキリ,朝は味噌汁だよね,今日も頑張る"
    },
    {
        "utterance":"こんにちは",
        "response":"ハロー",
        "options":"お腹減った,午後もがんばろう"
    }
]
```
カスタマイズした発話が応答として返される際、応答に対する発話候補として利用が可能になりました。


### v.0.50 (2020.3)
ベータ版を公開しました。

## Chaplus APIとは？
サービスに会話機能を手軽に導入することができるWeb API。チャットボットを始めとする『対話型』サービスに、雑談機能やQ&A機能を手軽に実装することができるAPIです。Chaplusは「Chat + Plus」を組み合わせた造語。

[公式サイト](https://www.chaplus.jp)

[DEMOページ](https://www.chaplus.jp/demo)

## 利用用途
チャットボットなどの対話型のサービスでは、ユーザからの想定外の問いかけや意図しない言葉を投げかけられることが当たり前。Chaplusはサービスの『雑談力』を向上させ、コミュニケーションの幅を広げます。

## 機能
### 1. 自然言語の発話に対する応答文を取得

ユーザの自然言語による発話に対し、複数の応答文の候補をScore付きで取得できます。慣用表現からニッチな内容まで幅広い話題に対応しています。Scoreは、ユーザ発話に対する応答文の「尤もらしさ」を表します。Scoreは0~1の範囲で計算され、1に近づけば近づくほど精度の高い応答とみなされます。

```
例:
ユーザ発話: 「仕事終わりのビールは最高」

応答候補

1位(Score: 0.8846)　適量でね
2位(Score: 0.8846) 今日も1日おつかれさま
3位(Score: 0.8846) カンパーイ！
4位(Score: 0.8462) 今日１日<#USERNAME>さんががんばった証拠ですね。
5位(Score: 0.4038) 発熱でいきましょう
6位(Score: 0.4038) 偉い
6位(Score: 0.3888) 牛乳派です
```
[試しに話しかける](https://www.chaplus.jp/demo)

### 2. 発話を柔軟にカスタマイズ

発話に対する応答文は特にカスタマイズせずに取得ができますが、特定の発話に対する応答を自由に設定することができます。Chaplus APIでは、カスタマイズしたい発話をJson形式で指定します。

```json
"utterancePairs":[
    {
        "utterance":"おはよう！",
        "response":"今日も1日頑張って！",
        "options":"顔洗ってスッキリ,朝は味噌汁だよね,今日も頑張る"
    },
    {
        "utterance":"こんにちは",
        "response":"ハロー",
        "options":"お腹減った,午後もがんばろう"
    }
]
```
カスタマイズした発話に関しては、APIが標準で返す応答よりも優先して利用されます。

### 3. NGワードの登録

APIの応答に含めるべきでないワードがある場合は、NGワードを指定できます。APIのリクエスト時に文字列の配列を渡すことができ、指定されたワードが応答に含まれた場合はAPIのレスポンスからその応答は除外されます。

### 4. 話者情報の登録

Chaplus APIでは、応答するAPIの話者を「エージェント(Agent)」と呼び、エージェントの情報はAPIリクエスト時に設定が可能です。現在設定が可能な項目は下記の通りです。

```json
"agentState":{
    "agentName":"エージェントの名前",
    "age":"エージェントの年齢",
    "tone":"エージェントの口調 (normal / kansai / koshu / dechu)"
}
```

APIから返される応答にエージェントの名前を含む場合は、設定した情報によって動的に名前や年齢が変更されます。口調は、「normal」(標準)か「kansai」（関西弁風）、「koshu」（甲州弁風）、「dechu」(デチュ語)を現在利用ができます。例えば「kansai」を設定している場合は、関西弁のような口調に変更されます。


# 利用方法

## 1. 登録

[登録ページ](https://www.chaplus.jp) より利用規約の確認と同意の上、APIの利用登録を実施します。

## 2. APIを利用する

現在、Chaplusは下記のエンドポイントを用意しています。

- [雑談応答API](/ChatAPI.md)

エンドポイントの仕様については上記のそれぞれのDocをご参照ください。


# 利用上の諸注意
[利用規約](https://www.chaplus.jp/agreement)をご確認の上、APIをご利用ください。
また、本APIはβ版です。β版は利用規約をご確認の上、非商用の目的でのみご利用ください。（商用利用や利用形態についてのご相談は下記の「問い合わせ」の連絡先までご連絡ください。）

## リクエスト時の諸注意
リクエストは１つのAPIキーにつき**1000リクエスト/日**までとさせていただきます。また、1秒あたり1回を超えるリクエストはお控えください。本APIはβ版につき、APIキーのリクエスト上限数を変更する可能性がございます。ご理解のほどよろしくお願いいたします。

## 著作情報
本APIは形態素解析処理に下記ライブラリを利用させていただいています。
[Kagome](https://github.com/ikawaha/kagome)
```
Kagome is licensed under the Apache License v2.0 and uses the MeCab-IPADIC, UniDic dictionary/statistical model. See NOTICE.txt for license details.
```

本APIの一部の応答は、下記をファインチューニングして構築した言語モデルをもとに応答を返しています。
[japanese-gpt2](https://github.com/rinnakk/japanese-gpt2)
```
MIT License

Copyright (c) 2021 rinna Co.,Ltd.

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

```

## 問い合わせ
お問い合わせは下記のメールアドレスまでお寄せください。

chaplus.ai@gmail.com
