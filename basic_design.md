# 観光地紹介Webアプリ 基本設計
チーム6　太田 朱音 / 高木 壱哲 / キム ジュヨン
## 画面設計
### 画面レイアウト
#### 管理者画面（Dashboard）
![alt text](src/管理者画面（Dashboard）.png)

#### 記事投稿画面
![alt text](src/記事投稿画面.png)

#### ユーザ属性分析画面
![alt text](src/ユーザ属性分析画面.png)

#### SNS投稿画面
![alt text](src/SNS投稿画面.png)

#### Top画面（QRコードからアクセス）
![alt text](src/Top画面(QRアクセス用).png)


### 画面遷移
 
#### 画面項目一覧
**管理者画面 #1**

-	地域別の訪問人数を地図上に可視化
-	人気記事のランキング表示 
-	新規記事投稿ボタン
    -	新規記事投稿 #2の画面に遷移
-	訪問属性分析
    -	訪問属性分析 #3の画面に遷移
-	SNS一括投稿ボタン
    -	SNS一括投稿 #4の画面に遷移

**新規記事投稿画面 #2**

-	記事タイトル入力欄
-	記事本文入力欄
-	カテゴリ選択プルダウン
-	画像アップロードボタン
-	公開・下書き切替スイッチ
-	下書きボタン
-	公開ボタン
    -	投稿成功時：OKの画面（「記事の投稿に成功しました」などのメッセージを表示）
    -   投稿失敗時：エラーメッセージを表示し、新規記事投稿画面 #2 に留まる
-	公開してSNSに投稿ボタン
    -	投稿成功時：
        - 	OKの画面（「記事の投稿に成功しました」などのメッセージを表示）
        -	SNS一括投稿画面 #4に遷移
    - 投稿失敗時：エラーメッセージを表示し、新規記事投稿画面 #2 に留まる

**ユーザ属性分析画面 #3**

**項目**

-	ヘッダー（共通ナビ）
-	６つの分析カード（全て表示用）
    -	AI要約
    -	観光客の人気訪問ルート
    -	再訪問率
    -	menu（スクロールの位置・ボタン？３枚）
    -	訪問者
    -	その他グラフサムネイル

**SNS投稿画面 #4**

**項目**

-	投稿したい記事［必須／ドロップダウン］
-	投稿したい SNS［必須／多選択チップ］
    -	X (Twitter) / Instagram / Facebook / Threads / LINE / メルマガ
-	投稿文［必須／複数行テキスト］
-	Preview ボックス（自動生成）
-	チェックボックス
    -	「投稿内容に誤りがないことを確認しました」
-	投稿する ボタン

**バリデーション**

-	投稿したい記事：未選択 NG
-	SNS：１件以上選択必須
-	投稿文：0〜200 文字（全 SNS 共通上限）
-	チェックボックス ON でないと送信不可
-	画像添付 UI は今回未実装（Figma に無し）

**SNS 一括投稿 #1（完了ダイアログ）**

-	「投稿が完了しました」
-	閉じるボタン → ダッシュボードへ戻る

**SNS 一括投稿#2 (エラー画面)**

-	各 SNS API からエラー戻り
    -	失敗した SNS 名を列挙
    -	「再試行」／「キャンセル」ボタン

**ユーザ属性分析画面 #5**

**項目**

-	ヘッダー（共通ナビ）
-	６つの分析カード（全て表示用）
    1.	AI要約
    2.	観光客の人気訪問ルート
    3.	再訪問率
    4.	menu（スクロールの位置・ボタン？３枚）
    5.	訪問者
    6.	その他グラフサムネイル

**Top画面 #6**

**項目**

-	ヘッダー：サイト名・ハンバーガーメニュー
-	観光地情報セクション
    -	メイン画像（163×250）
    -	(観光地の説明) テキスト
    -	現在地ラベル
-	周辺のお勧め観光スポット（画像 358×250＋ピンアイコン２つ）
-	イベントや企画の情報（小カード 3 枚）
-	観光地レビュー
    -	タイトル＋本文エリア（360×136）
    -	送信ボタン

**バリデーション**

-	レビュー送信
    -	タイトル・内容：空欄不可、各 200 文字以内
    -	送信時通信エラーはトースト表示


## 外部インターフェース
### APIリスト一覧
#### 記事公開ボタン押下
##### POST /articles/

**Request**

- article_id: id[]
- title：string[0, 150]
- body：string
- place_id: id[]
- category_ids: id[]
- tag_ids: id[]
- article_url_slug: string[1, 200]
- image_url: string(S3 Url)
- published: boolean, default: false

**Response**

- 200 OK
- 409 Conflict

##### POST /articles/publish
複数のアーティクルを選択してsnsチャネルに一括して共有する。

**Request**

- article_ids：id[] // ids of the target article to upload to a multitude of sns channels
- channels: '<‘x’ | ‘insta’ | ‘fb’ | ‘threads’ | ‘line’ | ‘mail’>[] // e.g. [‘insta’, ‘mail’]'
- description：string // 投稿文

**Response**

- 200 OK
- 400 Bad request
    - Requestのbodyが間違っている時


##### POST /tourist/language
QRからアクセスされたユーザー(観光客)の言語をアップデートし、その情報をDBにアップロードする。

**Request**

- lang： <ISO 639-1 codes> // e.g. ‘en’, ‘ja’, ‘ko

**Response**

- 200 OK
    - JSONのSPAの言語ファイル
- 400 Bad request
    - Requestのbodyが間違っている時

##### POST /tour/tourist/reviews
ユーザー(観光客)からのレビューをDBにアップロードする。

**Request**

- title: string //
- body： string //

**Response**

- 200 OK
- 400 Bad request
    - Requestのbodyが間違っている時

## データモデル設計
### テーブル設計
![alt text](src/svgviewer-output.svg)

