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

## 演習：サイト名を変更して CI/CD を体験する

ここからが本番です。**サイト名を「Web販 AWS勉強会サイト」から「DXCインターンシッププログラム」に変更**し、
push するだけで自動的に公開サイトへ反映される一連の流れ（CI/CD）を体験します。

```
Step 3  コードを変更         ← サイト名を書き換える
   │
Step 4  commit して push      ← 変更を GitHub に送る（自動化のトリガー）
   │
   ▼  ここから下は GitHub Actions が自動でやってくれる（CI/CD のキモ）
   │   web/ をビルドして Pages にデプロイ
   ▼
Step 5  公開サイトに反映       ← ブラウザで確認
```

---

## Step 3 — サイト名を変更する

`web/index.html` を Codespaces のエディタで開き、**3 か所**ある
`Web販 AWS勉強会サイト` を `DXCインターンシッププログラム` に書き換えます。

### ① ブラウザのタブに出るタイトル（`<title>`）

```html
<title>Web販 AWS勉強会サイト</title>
```
↓
```html
<title>DXCインターンシッププログラム</title>
```

### ② ページ上部の大見出し（`<h1>`）

```html
<h1>Web販 AWS勉強会サイト</h1>
```
↓
```html
<h1>DXCインターンシッププログラム</h1>
```

### ③ ページ下部の著作権表示（フッター）

```html
<p>© 2025 Web販 AWS勉強会サイト. All rights reserved.</p>
```
↓
```html
<p>© 2025 DXCインターンシッププログラム. All rights reserved.</p>
```

> 💡 VS Code（Codespaces）の **検索・置換**（`Ctrl + H`）を使うと、
> `Web販 AWS勉強会サイト` を一括で `DXCインターンシッププログラム` に置き換えられます。

変更したら **保存**（`Ctrl + S`）します。

---

## Step 4 — commit して push する（変更を送り出す）

Codespaces は最初から自分のリポジトリに接続・認証済みなので、そのまま push できます。

### 方法A：ターミナル

```bash
git add web/index.html
git commit -m "サイト名をDXCインターンシッププログラムに変更"
git push origin master
```

### 方法B：ソース管理パネル（ターミナルが苦手な人向け）

1. 左の **「ソース管理（Source Control）」** アイコンをクリック
2. `index.html` の **「＋」** で変更をステージ
3. メッセージ欄に `サイト名をDXCインターンシッププログラムに変更` と入力して **「Commit」**
4. **「Sync Changes」** をクリックして push

> この push が、次の自動デプロイ（CI/CD）の **トリガー** になります。

---

## Step 5 — 自動デプロイと公開を確認する

1. リポジトリの **「Actions」** タブをクリック
2. 今 push した内容で **「GitHub Pages へ自動デプロイ」** が動き出します
3. クリックして、各ステップが ✅ になるのを見守ります（1〜2 分）
4. 完了したら、公開URLを開きます：

```
https://<自分のユーザー名>.github.io/aws-handson-for-internship/
```

**ブラウザのタブ・大見出し・フッターが「DXCインターンシッププログラム」に変わっていれば成功です** 🎉

> 自分は「コードを書き換えて push しただけ」なのに、ビルドと公開が自動で行われました。
> これが **CI/CD（継続的インテグレーション / 継続的デリバリー）** の基本的な流れです。

---

## ふりかえり：今体験した CI/CD

| 段階 | やったこと | 担当 |
|---|---|---|
| 変更 | サイト名を書き換えて commit | あなた |
| push（トリガー） | `git push` で GitHub に送る | あなた |
| ビルド & デプロイ | `web/` を Pages 用にまとめて公開 | GitHub Actions（自動） |
| 公開 | 公開URLに反映 | GitHub Pages（自動） |

> 手作業なら「ファイルをサーバーにアップロード」していた部分を、**push をきっかけに自動化**したのが CI/CD です。
> 次に色やテキストを変えて push すれば、同じ流れで何度でも自動公開されます。

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
