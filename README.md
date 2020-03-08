# Chaplus API β
## Chaplus APIとは？
サービスに雑談機能を手軽に導入することができるWeb API。チャットボットを始めとする『対話型』サービスに、雑談機能を手軽に実装することができるAPIです。Chaplusは「Chat + Plus」を組み合わせた造語。

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
        "response":"今日も1日頑張って！"
    },
    {
        "utterance":"こんにちは",
        "response":"ハロー"
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

## 2. WEB API 仕様

### リクエスト

#### エンドポイント
https://www.chaplus.jp/v1/chat?apikey=APIKEY 

API登録時に取得したAPIKEYをクエリパラメータに付与します。


#### リクエストヘッダー

|  Key  |  Value  |
| ---- | ---- |
|  Content-Type  |  application/json  |

#### リクエストボディ
Jsonでリクエストを記述します。

##### リクエストパラメータ

|  パラメータ  |  説明  |  型  |  必須  |
| ---- | ---- | ---- | ---- |
|  utterance  |  ユーザの発話  | String | ◯ |
|  username  |  ユーザの名前(呼び名)  | String |  |
|  agentState  |  エージェントの情報  | agentStateオブジェクト |  |
|  addition  |  追加情報  | additionオブジェクト |  |
 

##### agentState
agentStateでは下記を指定。応答に含まれる置き換え文字が設定した情報に書き換わります。 

|  agentState  |  説明  | 置き換え記号 | 型 |
| ---- | ---- | ---- | ---- |
|  agentName  |  エージェントの名前  | <#NAME> | String |
|  tone  |  エージェントの口調(normal or kansai)  |  | String |
|  age  |  エージェントの年齢  | <#AGE> | String |

##### addition
additionでは応答をカスタマイズする設定が可能です。

|  addition  |  説明  | 型 |
| ---- | ---- | ---- |
|  options  |  ユーザに示す発話候補  | String配列 |
|  utterancePairs  |  発話ペアの配列  | utterancePairオブジェクトの配列 |
|  ngwords  |  NGワードのリスト  | String配列 |

##### utterancePair
utterancePairオブジェクトを利用して発話ペアを定義できます。

|  utterancePair  |  説明  | 型 |
| ---- | ---- | ---- |
|  utterance  |  ユーザの発話  | String |
|  response  |  ユーザの発話に対する応答  | String |

utterancePairで指定された発話ペアは、Chaplus APIが行う応答と同じロジックで処理されます。登録された発話と近しい表現の発話がリクエストされた際、応答としてresponseを利用します。(表記揺れの考慮あり)

サンプル
```json
{
    "utterance": "調子はどう？",
    "username": "太郎",
    "agentState":{
        "agentName": "エージェント",
        "tone": "kansai",
        "age": "14歳"
    }
    "addition" {
        "options":[
            "疲れた",
            "肩凝った"
        ]
        "ngwords" [
            "ため息",
            "やめてしまえ"
        ]
        "utterancePairs" [
            {
                "utterance": "肩凝った",
                "response":"適度に運動しないとね"
            },
        ]
    }
}
```

curl サンプル
```
curl -v -H "Content-Type: application/json" -X POST -d '{"utterance":"調子はどう？","username":"太郎","agentState":{"agentName":"エージェント","tone":"kansai", "age":"20歳"},"addition":{"options":["疲れた","肩凝った"],"utterancePairs":[{"utterance":"肩凝った","response":"適度に運動しないとね"}]}}' https://www.chaplus.jp/v1/chat\?apikey\=<APIKEY>
```


### ステータスコード

|  ステータス  |  説明  |
| ---- | ---- |
|  200  |  リクエスト成功  |
|  400 Bad Request  |  リクエストに問題  |
|  401 Unauthorized  |  認証エラー  |
|  404 Not Found  |  存在しないエンドポイント  |
|  500 Internal Server Error  |  サーバーのエラー  |

サンプル
```json
{
    "status": "Unauthorized Error",
    "message": "invalid key"
}
```


### レスポンス
#### Json

|  Key  |  説明  |  型  |
| ---- | ---- | ---- |
|  bestResponse  |  ユーザ発話に対する最もScoreの高い応答  |  responseオブジェクト  |
|  responses  |  ユーザ発話に対する応答候補  |  responseオブジェクトの配列  |
|  tokenized  |  分かち書きの結果 |  String配列  |
|  options  |  次の発話候補 |  String配列  |

optionsには、次にユーザが発話する際の参考になる「発話候補」が返されます。
optionsの内容は、ユーザ発話やAPIの応答によって決まる内容と、時間帯によって決まる内容、APIリクエストでカスタマイズした内容の３種類が含まれます。

##### response
responseオブジェクトは以下の要素を含みます。

|  response  |  説明  |  型  |
| ---- | ---- | ---- |
|  utterance  |  応答の発話文  |  String  |
|  score  |  応答のScore(どれだけ尤もらしい応答か)  | Float  |
|  url  |  応答と対応するURL(Defaultは空) |  String配列  |


サンプル
```json
{
    "utterance": "仕事終わりのビールは最高",
    "bestResponse": {
        "utterance": "適量でね",
        "score": 1,
        "url": ""
    },
    "responses": [
        {
            "utterance": "適量でね",
            "score": 1,
            "url": ""
        },
        {
            "utterance": "今日も1日おつかれさま",
            "score": 1,
            "url": ""
        },
        {
            "utterance": "今日１日太郎はんががんばった証拠やね。",
            "score": 0.7083,
            "url": ""
        },
        {
            "utterance": "カンパーイ！",
            "score": 0.7083,
            "url": ""
        },
        {
            "utterance": "牛乳派です",
            "score": 0.3333,
            "url": ""
        },
        {
            "utterance": "偉い",
            "score": 0.3333,
            "url": ""
        }
    ],
    "tokenized": [
        "名詞,サ変接続,*,*,*,*,仕事,シゴト,シゴト",
        "動詞,自立,*,*,五段・ラ行,連用形,終わる,オワリ,オワリ",
        "助詞,連体化,*,*,*,*,の,ノ,ノ",
        "名詞,一般,*,*,*,*,ビール,ビール,ビール",
        "助詞,係助詞,*,*,*,*,は,ハ,ワ",
        "名詞,一般,*,*,*,*,最高,サイコウ,サイコー"
    ],
    "options": [
        "疲れた",
        "肩凝った",
        "会社の人間関係は家まで持ち込みたくないですね",
        "今日は大変な１日だったなー",
        "明日はいい朝を迎えたいね",
        "ヒーリング音楽流すと安らぎますよね"
    ]
}
```

# 利用上の諸注意
[利用規約](https://www.chaplus.jp/agreement)をご確認の上、APIをご利用ください。
また、本APIはβ版です。利用規約をご確認の上、非商用の目的でのみご利用ください。（商用利用や利用形態についてのご相談は下記の「問い合わせ」の連絡先までご連絡ください。）

## リクエスト時の諸注意
リクエストは１つのAPIキーにつき**1000リクエスト/日**までとさせていただきます。また、1秒あたり1回を超えるリクエストはお控えください。本APIはβ版につき、APIキーのリクエスト上限数を変更する可能性がございます。ご理解のほどよろしくお願いいたします。

## 著作情報
本APIは形態素解析処理に下記ライブラリを利用させていただいています。
[Kagome](https://github.com/ikawaha/kagome)
```
Kagome is licensed under the Apache License v2.0 and uses the MeCab-IPADIC, UniDic dictionary/statistical model. See NOTICE.txt for license details.
```

## 問い合わせ
お問い合わせは下記のメールアドレスまでお寄せください。

chaplus.ai@gmail.com
