# locapos api document

## 概要

このドキュメントはlocapos.comへ接続するアプリケーションを作成するためのドキュメントです。このドキュメントの内容は今後予告なく変更される可能性があります。

## Client IDs

現時点で自動的にClientIDを払い出す仕組みはありません。twitter:tmyt あたりにコンタクトを取ってください。

## oauth

### GET /oauth/authorize

> OAuth2ベースユーザ認証

| | name | descriptoin |
|----|----|----|
| * | response_type | 必ず"token" |
| * | client_id | クライアントごとのIDを指定すること |
| * | redirect_uri | 認証に成功した際に転送されるURL|
|   | state | 認証時にクライアントから渡す任意の値 |

#### response

次の値をapplication/x-www-form-urlencoded 形式でFragment に渡します。

| | name | description |
|---|---|---|
| * |access_token||
| * |token_type| 常に "bearer" |
|   |state| リクエスト呼び出し時に与えられたstate引数の値 |

## locations

### POST /api/locations/update &#x1f512;

 > 位置情報の送信。

||name|description|
|---|---|---|
|*|latitude||
|*|longitude||
||heading|省略時については備考参照|
||posMode|GPS測位モード|
||private|true or false。省略するとfalse|
||key| /api/groups/new で生成したID |

#### response

> ok

#### 備考
privateとkeyの組み合わせで最大2面の地図へ位置情報がプロットされる。
プロットされる地図は以下の条件で決定される。

- private=trueのとき
  * 公開地図にプロットされない
  * ユーザ固有の暗黙的なグループへプロットされる
- private=falseのとき
  * 公開地図にプロットされる
  * ユーザ固有の暗黙的なグループへプロットされない
- keyが設定されていない場合
  * 任意グループへプロットされない
- keyにgoups/new で生成されたIDが設定されている場合
  * 任意グループへプロットされる
- keyにgroups/new で生成されたID以外が設定されている場合
  * 403エラーを返す

headingパラメタを省略した場合、サーバ側に前回位置情報がある場合についてのみheadingを自動計算する。
ただし、位置情報は送信後5分で自動削除されるため、送信間隔が5分以上ある場合には一切計算されない。
自動計算できなかった場合のheadingの値は0として扱われる。

また、headingの値は0～360の間へ自動的に正規化される。

### POST /api/locations/delete &#x1f512;

> 地図上から位置情報を削除する

||name|description|
|---|---|---|
||key|位置情報を削除するマップID|

#### response

> ok

#### 備考

地図上の位置情報は通常updateから5分後に自動で削除されます。このAPIは5分をまたずに位置情報を削除する場合に使用します。
削除するにする位置情報についてはkeyパラメタによって以下の条件で決定されます。

- key 未指定
     - private=falseで送信された位置情報が全体MAPから削除される。
- key=[hash]
     - private=true または、key=[hash] で送信された位置情報が指定したハッシュにマッチする地図から削除される
- key=*
      - 送信された位置情報のパラメタにかかわらず、すべての地図から位置情報が削除される

## groups

### GET /api/groups/join

>  グループへ参加するURLへリダイレクトする
> `locapos-api:///join?key=xxx`

||name|description|
|---|---|---|
|*|key|アプリケーションへ渡したいグループハッシュ|

### GET /api/groups/new

> 新しいグループハッシュを生成する

#### response

```javascript
{"key": "group-hash"}
```

## users

### GET /api/users/show &#x1f512;

> 現在アクティブなユーザ一覧を取得

||name|description|
|---|---|---|
||key||

#### response

```javascript
[{"provider":"provider-name","id":"000000","name":"screen-name","latitude":35,"longitude":135,"heading":40}]
```

### GET /api/users/me &#x1f512;

> 自分自身の情報を取得

#### response

```javascript
{"provider":"provier-name","id":"0000000","name":"screen-name"}
```

### GET /api/users/share &#x1f512;

> 自分自身の暗黙的なグループハッシュを取得する

#### response

```javascript
{"key": "group-hash"}
```

### POST /api/users/update &#x1f512;

> 自分自身の情報をアップデートする

||name|description|
|---|---|---|
||screen_name|Webサイト上の表示名|

#### response

> ok

#### 備考

`screen_name`に空文字列を指定してリクエストした場合、認証プロバイダから得られたデフォルトの表示名を復元します。

## map

### GET /[token]

||name|description|
|---|---|---|
||center|lat,lon 形式|
||zoom||

#### 備考

locapos.com/[hash] でハッシュを指定すると表示する地図が指定できます。
この場合のハッシュには、groups/new で生成したハッシュや、users/share で生成したハッシュが必要です。

users/share のハッシュではprivate=true に指定している特定のユーザのみ、
gourps/new のハッシュではkey=[hash] でプロット先グループを指定しているユーザのみが表示される。
ハッシュが指定されていない場合は、private=true でないユーザすべてが表示されます。

## notes

### 認証トークンの有効期限

認証トークンは最後のAPI呼び出しから1か月後に削除され、クライアントは再度認証が必要になります。
