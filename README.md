# NextToCopilotApp

Next.jsのHello Worldアプリケーションを3つの実行環境で動作させる最小構成のPoCです。

## このPoCの目的

同じNext.jsアプリケーションコードが、以下の3環境で動作できることを確認します。

- ローカル環境
- Google Cloud Run
- GitHub Copilot App

機能はトップページに "Hello World!" を表示するだけの最小構成です。

## 技術構成

- Next.js（最新安定版 / App Router / TypeScript）
- npm
- `output: "standalone"` によるCloud Run向けビルド

## ローカルでの起動方法

```bash
npm ci
npm run dev
```

http://localhost:3000 にアクセスすると "Hello World!" が表示されます。

## Dockerでの起動方法

```bash
docker build -t next-to-copilot-app .
docker run --rm -p 8080:8080 next-to-copilot-app
```

http://localhost:8080 にアクセスすると "Hello World!" が表示されます。

## Google Cloud Runへのデプロイ例

Artifact Registryを個別に操作せず、ソースから直接デプロイします。

```bash
gcloud run deploy next-to-copilot-app \
  --source . \
  --region asia-northeast1 \
  --allow-unauthenticated
```

Cloud Runはコンテナに `PORT` 環境変数（既定で8080）を渡します。Next.jsのstandaloneサーバーは `PORT` を参照して待ち受けるため、追加設定は不要です。

## GitHub Copilot Appでの起動方法

GitHub Copilot App用の設定は `.github/github-app.yml` に定義しています。

- セッション開始時に `npm ci` で依存関係をインストールします
- `npm run dev` でNext.jsの開発サーバーを起動します
- Next.jsが出力する起動ログからローカルサーバーのURL（ポート3000）を検出します
- サーバー起動後にintegrated browserが自動的に開きます

## 3つの環境で同じアプリケーションコードを利用していること

`app/layout.tsx` と `app/page.tsx` のアプリケーションコードは3環境で共通です。Next.js本体のコードにはGitHub Copilot App専用の分岐や依存関係を含めていません。

## 環境ごとの差分がどこにあるか

環境差分は実行方法と設定ファイルに閉じ込めています。

| 環境 | 実行方法 | 差分を持つファイル |
| --- | --- | --- |
| ローカル | `npm run dev` | なし（package.jsonのscriptsのみ） |
| Google Cloud Run | Dockerコンテナ（standaloneビルド） | `Dockerfile` / `.dockerignore` / `next.config.ts` の `output: "standalone"` |
| GitHub Copilot App | `npm ci` + `npm run dev` の自動実行 | `.github/github-app.yml` |
