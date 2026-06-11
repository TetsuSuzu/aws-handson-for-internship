# Session4 — GitHub Actions でサイトを自動公開する

## このセッションで学ぶこと

**GitHub Actions** を使って、`web/` を編集して push すると
**自動で GitHub Pages に公開** される仕組みを体験します。**AWS は不要**です。

```
PCで web/ を編集
  │  git push
  ▼
GitHub（自分の Fork したリポジトリ）
  │  GitHub Actions が自動起動
  ▼
┌─────────────────────┐
│   GitHub Actions    │  ← web/ をビルド＆アップロード
└─────────────────────┘
  │
  ▼
┌─────────────────────┐
│   GitHub Pages      │  ← 公開URLでサイトが見られる
└─────────────────────┘
   https://<ユーザー名>.github.io/aws-handson-for-internship/
```

---

## GitHub Actions の用語

| 用語 | 意味 |
|---|---|
| **Workflow（ワークフロー）** | 自動化の手順書。YAML ファイルで書く（`.github/workflows/deploy-pages.yml`） |
| **Trigger（トリガー）** | 起動のきっかけ（今回は `web/` への push） |
| **Job / Step** | 処理のまとまり / 個々の処理 |
| **Runner（ランナー）** | 処理を実行する GitHub のサーバー |

---

## Step 1 — GitHub Pages を有効にする（最初に1回だけ）

自動公開を動かす前に、**自分の Fork したリポジトリ**で GitHub Pages を有効にします。

1. リポジトリの **「Settings」** タブを開く
2. 左メニュー **「Pages」** をクリック
3. **「Build and deployment」→「Source」** で **「GitHub Actions」** を選択

> 「Deploy from a branch」ではなく **「GitHub Actions」** を選ぶのがポイントです。
> この設定をしないと、初回のワークフローが `Create Pages site failed`（権限エラー）で失敗します。
> （ワークフローにも自動有効化の設定を入れていますが、Fork したリポジトリでは
> 権限の都合で自動有効化できないため、この手動設定が必要です）

---

## Step 2 — ワークフローを確認する

`.github/workflows/deploy-pages.yml` が自動公開の設定ファイルです。
中身は次のようになっています（編集不要）。

```yaml
on:
  push:
    branches:
      - master
    paths:
      - 'web/**'        # web/ が変わったときだけ起動

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v5
        with:
          enablement: true         # Pages が未設定なら自動で有効化
      - uses: actions/upload-pages-artifact@v3
        with:
          path: web                 # web/ フォルダを公開
      - uses: actions/deploy-pages@v4
```

- `web/` を変更して push したときだけ動きます。
- `web/` フォルダの中身をそのまま GitHub Pages に公開します。

---

## Step 3 — 変更して push する

`web/index.html` か `web/styles.css` を少し編集して、GitHub に push します。
Codespaces は最初から自分のリポジトリに接続・認証済みなので、そのまま push できます。

Codespaces のターミナルで：

```bash
git add web/
git commit -m "サイトを更新（GitHub Actions の動作確認）"
git push origin master
```

> ターミナルが苦手な人は、左の **「ソース管理（Source Control）」** アイコンからでもOKです。
> 変更ファイルの「＋」でステージ → メッセージを入力して **「Commit」** → **「Sync Changes」** で push できます。

---

## Step 4 — 公開を確認する

1. リポジトリの **「Actions」** タブをクリック
2. 実行中／完了したワークフロー **「GitHub Pages へ自動デプロイ」** をクリック
3. 各ステップが ✅ になれば成功
4. **「Settings」→「Pages」** に表示される公開URLを開く：

```
https://<自分のユーザー名>.github.io/aws-handson-for-internship/
```

自分が編集した内容がインターネット上で見られれば完成です 🎉

> 初回は反映まで 1〜2 分ほどかかることがあります。

---

## 補足：公開サイトでも API は動く？

`web/index.html` に設定したエンドポイント（Session2/3）は、公開サイトでもそのまま動きます。
ただし、事務局の API 側で **公開URL（`https://<ユーザー名>.github.io`）からのアクセスを許可（CORS）** している必要があります。
うまく動かない場合は事務局に確認してください。

---

## よくあるエラーと対処

| 症状 | 原因 | 対処 |
|---|---|---|
| Actions が起動しない | `web/` 以外しか変更していない | `web/` の中のファイルを変更して push |
| ページが 404 | Pages の Source が未設定 | Step 1 で **「GitHub Actions」** を選択したか確認 |
| `Pages site failed` | Pages が無効 | Settings → Pages を一度開いて有効化 |
| 公開サイトで API がエラー | CORS 未許可 | 事務局に公開URLを伝えて許可してもらう |

---

🎉 これで全セッション完了です。おつかれさまでした！
