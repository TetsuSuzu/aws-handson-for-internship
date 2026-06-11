# インターンシップ向け Web ハンズオン

GitHub と GitHub Actions を使って、**ブラウザだけ**で Web サイトを作り、世界に公開するハンズオンです。
開発はクラウド上の **GitHub Codespaces** で行うため、**AWS アカウントも、PCへのインストールも不要**です。

> サーバー（S3・API Gateway・Lambda・DynamoDB・生成AI）は **事務局が事前に用意** しています。
> みなさんは、配布される **API エンドポイント** を使って、フロントエンド（Web 画面）の開発に集中します。

---

## ⚡ クイックスタート（Codespaces ワンクリック起動）

下のボタンから、ブラウザ上の開発環境（GitHub Codespaces）をすぐ起動できます。
Python・ポート転送・拡張機能はあらかじめ設定済みです（[.devcontainer/](.devcontainer/devcontainer.json)）。

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/TetsuSuzu/aws-handson-for-internship?quickstart=1)

> 💡 **受講生は、まず自分のアカウントに Fork してから起動してください。**
> Fork 後は **自分のリポジトリの「Code」→「Codespaces」** から起動すると、
> 編集内容を自分のリポジトリに push でき、GitHub Pages での公開（Session4）まで進められます。
> 上のボタンは事務局リポジトリをそのまま試す用です。詳しい手順は [Session0](Session0/README.md) を参照。

---

## このハンズオンで学べること

| テーマ | 内容 |
|---|---|
| Git / GitHub | リポジトリの Fork・clone・編集・commit・push |
| Web フロントエンド | HTML / CSS / JavaScript の編集とローカル実行 |
| API 連携 | 配布された REST API（会員登録・生成AI）をブラウザから呼び出す |
| CI/CD | GitHub Actions で push するだけで GitHub Pages に自動公開 |

---

## 完成イメージ

```
　　　ブラウザ（GitHub Codespaces）                  GitHub / クラウド
┌───────────────────────────┐        ┌──────────────────────────────┐
│  web/ を編集                       │        │  GitHub Actions                            │
│   ├ index.html                     │ push   │     └→ GitHub Pages に自動公開             │
│   ├ styles.css        ───────────▶ │        │         https://<ユーザー名>.github.io/... │
│   └ photo*.png                     │        │                                            │
│                                    │        │  ┌── 事務局が用意済み ─────────────┐       │
│  python -m http.server 8000        │  API   │  │ API Gateway → Lambda → DynamoDB │       │
│   → ポート転送URLで表示    ────────┼────────┼─▶│ API Gateway → Lambda → 生成AI    │       │
└───────────────────────────┘        │  │ S3（公開用バケット）             │       │
                                                │  └──────────────────────────────┘       │
                                                └──────────────────────────────┘
```

---

## セッション一覧

| Session | 内容 | キーワード |
|---|---|---|
| [Session0](Session0/README.md) | 環境セットアップ（GitHub アカウント・Fork・Codespaces 起動・サーバー実行） | GitHub / Codespaces |
| [Session1](Session1/README.md) | Web サイトを動かして HTML/CSS を編集する | HTML / CSS / ポート転送 |
| [Session2](Session2/README.md) | 会員登録 API を呼び出す（配布エンドポイントを設定） | REST API / fetch |
| [Session3](Session3/README.md) | AI おすすめ機能を呼び出す（配布エンドポイントを設定） | 生成AI / API |
| [Session4](Session4/README.md) | GitHub Actions で GitHub Pages に自動公開する | CI/CD / GitHub Actions |

---

## フォルダ構成

```
.
├── README.md                 ← このファイル
├── web/                      ← ★みなさんが編集する Web サイト本体
│   ├── index.html
│   ├── styles.css
│   ├── photo1.png
│   └── photo2.png
├── Session0〜4/README.md     ← 各セッションの手順書
├── .github/workflows/        ← GitHub Actions（自動公開の設定）
│   └── deploy-pages.yml
└── staff/                    ← 事務局向け：サーバー構築手順・コード（学生は使いません）
```

---

## 事前に必要なもの

| 必要なもの | 用途 |
|---|---|
| GitHub アカウント（無料） | リポジトリの Fork・Codespaces・サイト公開（GitHub Pages） |
| ブラウザ（Chrome / Edge など） | Codespaces での編集・サイトの表示・動作確認 |

> 開発は **GitHub Codespaces**（ブラウザ上の VS Code）で行うため、PC への VS Code・Python・Git のインストールは不要です。
> Codespaces には無料枠（個人アカウントで月60時間目安）があります。使い終わったら Codespace を **Stop** しておきましょう。

---

## まずはここから

➡️ **[Session0：環境セットアップ](Session0/README.md)** から始めてください。
