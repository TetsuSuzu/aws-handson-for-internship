# 事務局向け — バックエンド構築手順

このフォルダは **事務局（運営）向け** です。学生は使いません。

学生は AWS アカウントを持たない前提のため、以下のサーバー側リソースを
**事務局が事前に1セット構築し、エンドポイント URL を学生に配布** します。

| リソース | 用途 | 学生の使い方 |
|---|---|---|
| S3 バケット（公開用） | 学生の成果物の参考公開先（任意） | 通常は GitHub Pages を使うため必須ではない |
| API Gateway + Lambda + DynamoDB | 会員登録 API | Session2 で `registrationUrl` に設定 |
| API Gateway + Lambda + Bedrock | AIおすすめ API | Session3 で `bedrockUrl` に設定 |

> 学生は1つの共有 API を全員で呼び出します。登録データは全員分が同じ DynamoDB に入る前提です（ハンズオン用途）。

---

## このフォルダのファイル

| ファイル | 用途 |
|---|---|
| `lambda_users.py` | 会員登録 Lambda（DynamoDB へ保存） |
| `lambda_users_test_event.json` | 会員登録 Lambda のテストイベント |
| `lambda_bedrock.py` | AIおすすめ Lambda（Bedrock 呼び出し） |
| `bucket_policy.json` | S3 公開用バケットポリシー（任意・参考） |

---

## ★ 最重要：CORS 設定

学生は次の2つのオリジンから API を呼び出します。両 API Gateway で許可してください。

- **GitHub Codespaces のポート転送URL**：`https://<英数字>-8000.app.github.dev`（学生ごとに異なる）
- **GitHub Pages の公開URL**：`https://<ユーザー名>.github.io`（学生ごとに異なる）

学生ごとにオリジンが変わるため、ハンズオン用途では **CORS を `*`（全許可）にするのが簡単**です。
両 API の各リソースで **CORS を有効化**し、以下を設定します。

| 項目 | 値 |
|---|---|
| Access-Control-Allow-Origin | `*`（全許可。学生ごとにオリジンが変わるため個別指定は非現実的） |
| Access-Control-Allow-Methods | `POST`, `OPTIONS` |
| Access-Control-Allow-Headers | `Content-Type` |

> `*` を使う場合、ブラウザは Cookie 等の資格情報を送りません。今回の API は資格情報不要のため問題ありません。

---

## 1. 共通：IAM ロールを作成する

両 Lambda が使うロールを1つ作成します。

| 項目 | 値 |
|---|---|
| 信頼されたエンティティ | AWS のサービス → Lambda |
| 許可ポリシー | `AmazonDynamoDBFullAccess` / `AWSLambdaBasicExecutionRole` / `AmazonBedrockFullAccess` |
| ロール名 | 任意（例：`handson-lambda-role`） |

---

## 2. 会員登録 API（API Gateway → Lambda → DynamoDB）

### 2-1. DynamoDB テーブル

| 項目 | 値 |
|---|---|
| テーブル名 | `users` |
| パーティションキー | `id`（文字列） |
| ソートキー | なし |

### 2-2. Lambda 関数

| 項目 | 値 |
|---|---|
| 関数名 | 任意（例：`users-post-function`） |
| ランタイム | Python 3.12 |
| 実行ロール | 1. で作成したロール |
| コード | `lambda_users.py` を貼り付けてデプロイ |

テスト：`lambda_users_test_event.json` の内容でテスト実行 → `{"message": "登録が完了しました"}` が返れば OK。

### 2-3. API Gateway（REST API）

1. **REST API** を作成
2. `/`（ルート）に **POST** メソッドを作成
   - 統合タイプ：**Lambda 関数**
   - **Lambda プロキシ統合：オフ**
   - 対象 Lambda：`users-post-function`
3. **統合リクエスト** → マッピングテンプレート追加
   - コンテンツタイプ：`application/json`
   - テンプレート本文：`$input.json('$')`
4. **CORS を有効化**（上の「★最重要」参照）
5. **デプロイ**（ステージ名：例 `users-stage`）
6. 「URL を呼び出す」をメモ → **学生に配布（Session2 用）**

```
例: https://xxxxxxxxxx.execute-api.ap-northeast-1.amazonaws.com/users-stage/
```

> フロント（`web/index.html`）はリクエストボディをそのまま JSON で送るため、
> マッピングテンプレート `$input.json('$')` で Lambda の `event` に渡しています。

---

## 3. AIおすすめ API（API Gateway → Lambda → Bedrock）

### 3-1. Bedrock のモデルアクセス

- リージョン **us-east-1（バージニア北部）** で、使用するモデル（Claude Sonnet 4.6）のアクセスを有効化しておく。
- `lambda_bedrock.py` は推論プロファイル `global.anthropic.claude-sonnet-4-6` を使用。

### 3-2. Lambda 関数

| 項目 | 値 |
|---|---|
| 関数名 | 任意（例：`recommend-function`） |
| ランタイム | Python 3.12 |
| 実行ロール | 1. で作成したロール |
| タイムアウト | **1 分**（一般設定 → 編集） |
| コード | `lambda_bedrock.py` を貼り付けてデプロイ |

テスト：`{"key1": "長野でおすすめの料理を教えてください"}` で実行 → AI の回答テキストが返れば OK。

### 3-3. API Gateway（REST API）

1. **REST API** を作成
2. リソース（例：`/recommend`）を作成し、**POST** メソッドを作成
   - 統合タイプ：**Lambda 関数**
   - **Lambda プロキシ統合：オフ**
   - 対象 Lambda：`recommend-function`
3. **統合リクエスト** → マッピングテンプレート追加
   - コンテンツタイプ：`application/json`
   - テンプレート本文：`{"key1":$input.json('$.key1')}`
4. **CORS を有効化**（上の「★最重要」参照）
5. **デプロイ**（ステージ名：例 `prod`）
6. 「URL を呼び出す」＋リソース名をメモ → **学生に配布（Session3 用）**

```
例: https://xxxxxxxxxx.execute-api.ap-northeast-1.amazonaws.com/prod/recommend
```

> API Gateway のタイムアウトは最大 29 秒です。Bedrock の応答が遅いとエラーになることがあります。

---

## 4. （任意）S3 公開用バケット

学生は通常 **GitHub Pages** で公開するため必須ではありませんが、
事務局側で参考用に静的サイトをホストする場合の設定です。

1. バケットを作成（リージョン：ap-northeast-1）
2. 「パブリックアクセスをすべてブロック」を **オフ**
3. プロパティ → 静的ウェブサイトホスティングを **有効化**（インデックス：`index.html`）
4. アクセス許可 → バケットポリシーに `bucket_policy.json` を貼り付け
   （`Resource` のバケット名を実際の名前に置換）

---

## 学生への配布物チェックリスト

事前に学生へ案内・配布してください。

- [ ] Fork 元リポジトリの URL
- [ ] 会員登録 API のエンドポイント URL（Session2 用）
- [ ] AIおすすめ API のエンドポイント URL（Session3 用）
- [ ] 両 API の CORS が `*`（または `*.app.github.dev` と `*.github.io`）を許可していること

---

## 注意

- 会員登録 API は共有のため、テストデータが全員分混在します。必要に応じて定期的に `users` テーブルを掃除してください。
- ハンズオン終了後はコスト発生を避けるため、不要なリソース（API Gateway / Lambda / DynamoDB / Bedrock の設定）を削除してください。
