# CORSについて調べたことまとめ
## 同一オリジンポリシー
Same-Origin Policy  
異なるオリジン間のアクセスを制限する仕組み

- JSでのクロスオリジンへのリクエスト
- JSでのiframe内のクロスオリジンページへのアクセス
- Web StorageやIndexedDBに保存されたクロスオリジンデータへのアクセス

> JSでのiframe内のクロスオリジンページへのアクセス

悪意あるウェブサイトからブラウザー内のJSで機密性の高い情報を読み取り、攻撃者に中継することを防止できる  

### 許可されているケースもある
- scriptタグのsrc属性
- linkタグのhref属性
- imgタグのsrc属性
- formタグのaction属性

### オリジン
https://example.com:433/index.html
- https:// -> スキーム名
- example.com -> ホスト名
- :433 -> ポート番号
- /index.html -> パス名

上記における
> - https:// -> スキーム名
> - example.com -> ホスト名
> - :433 -> ポート番号

をオリジンという

https://example.com:433  
https://example.com:433  
同じオリジンを同一オリジン

https://example.com:433  
http://example2.com:80  
異なるオリジンをクロスオリジン

## CORSとは
Cross-Origin Resource Sharing  
オリジン間リソース共有  
クロスオリジンへのリクエストを可能にする仕組み  
厳密にはレスポンスにアクセスできないようにブラウザが制限している  
ブラウザがサーバーからのレスポンスのヘッダーに付与された「この相手ならアクセスしてもいいよ」を確認し、許可されたクライアントならレスポンスにアクセスできるようにし、そうでない場合はアクセスを禁止している

## 単純リクエスト
クロスオリジン間でもいくつかは、ブラウザがデフォルトで送信可能としているリクエストがある
- img, linkなどのGET
- formのGET、POST

より具体的な条件は下記の通り
- GET, HEAD, POSTのいずれかが使われている
- ユーザーエージェントによって自動で設定されるヘッダーが使われている
- もしくは、（手動設定された）Accept, Accept-Language, Content-Language, Content-Type, Rangeヘッダーが使われている
  - Content-Typeはapplication/x-www-form-urlencoded, multipart/form-data, text/plainのいずれか

## プリフライトリクエスト
単純リクエスト以外のリクエストで発生する事前問い合わせのリクエスト  
許可されている内容であればリクエストを送信する  
OPTIONメソッドが使用される

## なぜプリフライトリクエストが必要か
罠サイトがあり、その罠サイトには正規のサイトに対して何かしらのDELETEのリクエストを送るようにしていたとする  
正規サイトの資格情報を持っているユーザーが罠サイトに誘導され、DELETEリクエストのトリガーを実行した場合、何もないとそのままサーバー側へリクエストが到達してしまう

プリフライトリクエストがある場合、そのDELETEリクエストが事前に許可されたメソッドか、また接続元として許可されたオリジンかをプリフライトリクエストにて検証することができる

## CORSを有効にするには
下記のヘッダーを付与してレスポンスする
```
# アクセスを許可するオリジン
Access-Control-Allow-Origin: *
```

またプリフライトリクエストでは以下もレスポンスするようにする
```
# 利用可能なメソッド
Access-Control-Allow-Methods: GET, PUT
# 利用可能なヘッダ
Access-Control-Allow-Headers: Content-Type
# プリフライトリクエストのキャッシュ（ms）
Access-Control-Max-Age: 60
```
これを受け取ったブラウザがリクエスト内容と照らし合わせて実際にリクエストを送信するか決める

## 流れ
![CleanShot 2024-06-20 at 02 29 17@2x](https://github.com/tamo-tmry/free-book/assets/121565599/f663b8a5-bf79-472a-85be-4c930119ff7e)

