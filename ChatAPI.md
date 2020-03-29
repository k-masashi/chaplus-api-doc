# 雑談応答API
## APIの機能

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

## Web API仕様

### 実装サンプル

- [Slack Bot作成事例（GAS / JavaScript）](https://qiita.com/maKunugi/items/6a6abf83ca27716541df)
- [LINE Bot作成事例（GAS / JavaScript）](https://qiita.com/maKunugi/items/e6fc7f51071ab3696d5f)

### リクエスト
METHOD: POST

#### エンドポイント
https://www.chaplus.jp/v1/chat?apikey=APIKEY 

API登録時に取得したAPIKEYをクエリパラメータに付与します。

※ Rakuten Rapid APIからChaplusをご利用の方は、Rakuten Rapid APIの詳細ページにあるエンドポイントの仕様をご確認ください。


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
|  options  |  ユーザに示す発話候補  | String配列 |
|  utterancePairs  |  発話ペアの配列  | utterancePairオブジェクトの配列 |
|  ngwords  |  NGワードのリスト  | String配列 |
|  unknownResponses  |  高スコアの回答がなかった際に利用する発話  | String配列 |

##### utterancePair
utterancePairオブジェクトを利用して発話ペアを定義できます。

|  utterancePair  |  説明  | 型 |
| ---- | ---- | ---- |
|  utterance  |  ユーザの発話  | String |
|  response  |  ユーザの発話に対する応答(カンマ区切りで複数可)  | String |

utterancePairで指定された発話ペアは、Chaplus APIが行う応答と同じロジックで処理されます。登録された発話と近しい表現の発話がリクエストされた際、応答としてresponseを利用します。(表記揺れの考慮あり) responseに入れる値は、「,」で区切ることによって複数登録することができます。例えば、「こんにちは」に対して「ハロー,hello,こんちわ！」を登録した場合、３つの候補の内からランダムで応答が返されます。

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
                "response":"適度に運動しないとね,定期的に肩を回そう！"
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
    "utterance": "仕事終わりのビールは最高",
    "bestResponse": {
        "utterance": "適量でね",
        "score": 1,
        "url": "",
        "options": null
    },
    "responses": [
        {
            "utterance": "適量でね",
            "score": 1,
            "url": "",
            "options": null
        },
        {
            "utterance": "今日も1日おつかれさま",
            "score": 1,
            "url": "",
            "options": null
        },
        {
            "utterance": "今日１日太郎はんががんばった証拠やね。",
            "score": 0.7083,
            "url": "",
            "options": null
        },
        {
            "utterance": "カンパーイ！",
            "score": 0.7083,
            "url": "",
            "options": null
        },
        {
            "utterance": "牛乳派です",
            "score": 0.3333,
            "url": "",
            "options": null
        },
        {
            "utterance": "偉い",
            "score": 0.3333,
            "url": "",
            "options": null
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
