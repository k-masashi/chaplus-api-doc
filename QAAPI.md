# Q&A応答API
## APIの機能

Q&A自動応答を行うための機能を提供します。

```
例:
ユーザ発話: 「A号棟のレストランの料理はおいしいの？」

応答候補

1位(Score: 0.98)　栄養バランスの良く、美味しく食べられる食事が用意されていますよ。

次のユーザの発話候補
- 「A号棟のレストランの近くにトイレはありますか？」
- 「A号棟のレストランの営業時間は？」
```

Q&A機能をすばやく構築するための仕組みも提供します。質問と応答のペア、必要であればURLを指定して自動Q&Aシステムとして利用することが可能です。

```json
"qaSet": [
    {
        "tag": "A号棟",
        "question": "レストランの料理はおいしいですか？",
        "answer": "栄養バランスの良く、美味しく食べられる食事が用意されていますよ。",
        "options": [
            "A号棟のレストランの近くにトイレはありますか？",
            "A号棟のレストランの営業時間は？"
        ],
        "url": "http://example.com"
    }
]
```
## Web API仕様

### リクエスト
METHOD: POST

#### エンドポイント
https://www.chaplus.jp/v1/qa?apikey=APIKEY 

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
|  tone  |  エージェントの口調(normal,kansai,koshu,dechu)  |  | String |
|  age  |  エージェントの年齢  | <#AGE> | String |

##### addition
additionでは応答をカスタマイズする設定が可能です。

|  addition  |  説明  | 型 |
| ---- | ---- | ---- |
|  options  |  ユーザの発話候補  | String配列 |
|  qaSet  |  Q＆Aの発話ペア  | qaSetオブジェクト配列 |
|  unknownResponses  |  高スコアの回答がなかった際に利用する発話  | String配列 |

##### qaSet
qaSetオブジェクトを用いてQ&Aの発話ペアを定義できます。

|  qaSet  |  説明  | 型 |
| ---- | ---- | ---- |
|  tag  |  質問に関するタグ(必須)  | String |
|  question  |  質問(必須)  | String |
|  answer  |  回答(必須)  | String |
|  options  | 個別の回答の際にのみ利用したいユーザの発話候補  | String配列 |
|  url  |  個別の回答に紐づくurl  | String |

qaSetで指定された発話ペアは、Chaplus APIが行う応答と同じロジックで処理されます。登録された発話と近しい表現の発話がリクエストされた際、応答としてresponseを利用します。qaSetでもoptionsを指定できます。上の階層にあるoptionsとは異なり、Q&Aの個別の回答に紐づくoptionsになります。指定した回答が発話される際のみ利用されます。個別の回答に左右されずユーザに向けて提示したい候補は、上の階層にあるoptionsを利用しましょう。

サンプル
```json
{
    "tag": "A号棟",
    "question": "レストランの料理はおいしいですか？",
    "answer": "栄養バランスの良く、美味しく食べられる食事が用意されていますよ。",
    "options": [
        "A号棟のレストランの近くにトイレはありますか？",
        "A号棟のレストランの営業時間は？"
    ],
    "url": "http://example.com"
}
```

curl サンプル
```
curl -v -H "Content-Type: application/json" -X POST -d '{
    "utterance": "トイレどこですか？",
    "username": "太郎",
    "agentState": {
        "agentName": "エージェント",
        "tone": "kansai",
        "age": "2歳"
    },
    "addition": {
        "unknownResponses": [],
        "qaset": [
            {
                "tag": "A号棟",
                "question": "レストランの料理はおいしいですか？",
                "answer": "栄養バランスの良く、美味しく食べられる食事が用意されていますよ。",
                "options": [
                    "A号棟のレストランの近くにトイレはありますか？",
                    "A号棟のレストランの営業時間は？"
                ],
                "url":""
            }
        ]
    }
}' https://www.chaplus.jp/v1/qa\?apikey\=<APIKEY>
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
|  options  |  次の発話候補(個別のResponseによらない) |  String配列  |

optionsには、次にユーザが発話する際の参考になる「発話候補」が返されます。
optionsの内容は、ユーザ発話やAPIの応答によって決まる内容と、時間帯によって決まる内容、APIリクエストでカスタマイズした内容の３種類が含まれます。

##### response
responseオブジェクトは以下の要素を含みます。

|  response  |  説明  |  型  |
| ---- | ---- | ---- |
|  utterance  |  応答の発話文  |  String  |
|  score  |  応答のScore(どれだけ尤もらしい応答か)  | Float  |
|  url  |  応答と対応するURL(Defaultは空) |  String配列  |
|  options  |  個別のresponseに対する次の発話候補(DefaultはNULL) |  String配列  |


サンプル
```json
{
    "utterance": "栄養バランスの良く、美味しく食べられる食事が用意されていますよ。",
    "bestResponse": {
        "utterance": "栄養バランスの良く、美味しく食べられる食事が用意されていますよ。",
        "score": 1,
        "url": "https://example.hoge/hoge",
        "options": ["A号棟のレストランの営業時間は？"]
    },
    "responses": [
        {
            "utterance": "栄養バランスの良く、美味しく食べられる食事が用意されていますよ。",
            "score": 1,
            "url": "https://example.hoge/hoge",
            "options": ["A号棟のレストランの営業時間は？"]
        }
    ],
    "options": [
        "解決しなかったので、個別に問い合わせる",
        "A号棟にお風呂はありますか？",
    ]
}
```
