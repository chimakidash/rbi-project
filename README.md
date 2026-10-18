# Remote Browser Isolation (RBI) Project

Docker 上の Chromium を Nginx 経由で利用するための構成例です。20ユーザー分のブラウザーサービスを定義しています。

## ディレクトリ構成

```text
rbi-project/
├── .gitignore
├── README.md
├── docker-compose.yml
├── nginx.conf
└── policies/
	└── managed_policies.json
```

## 前提条件

- Docker Engine と Docker Compose プラグイン
- Docker ホストに、コンテナ用メモリのほかにホスト OS 用メモリがあること
- 80 番ポートが利用可能であること

## 起動

プロジェクトのルートで実行します。

```sh
docker compose config
docker compose up -d
docker compose ps
```

ブラウザーには `http://<DockerホストのIP>/user1/` のようにアクセスします。`user1` から `user20` まで利用できます。

停止する場合:

```sh
docker compose down
```

## 設定について

- Chromium の共通設定で、各サービスに CPU 1.0、メモリ 1.2 GB の上限を指定しています。
- 15 FPS などの帯域設定は構成例であり、実際の画質・帯域・同時利用数は環境と利用内容に応じた負荷試験が必要です。
- Compose は 443 番ポートも公開しますが、同梱の Nginx 設定は HTTP のみです。HTTPS を使う場合は TLS の待受と証明書設定を別途追加してください。
- この構成にはユーザー認証がありません。インターネットへ公開せず、利用ネットワークを制限してください。外部公開が必要な場合は、認証・TLS・アクセス制御を先に構成してください。

## GitHub への登録

GitHub CLI (`gh`) を使う場合は、GitHub にログインしたうえでプロジェクトのルートから実行します。

```sh
gh auth login
git init
git add .
git status
git commit -m "Initial RBI project configuration"
gh repo create rbi-project --private --source=. --remote=origin --push
```

`git status` で登録対象を確認してからコミットしてください。リポジトリを公開する場合は、設定・ネットワーク情報・証明書などの秘密情報が含まれていないことを確認し、`--private` を `--public` に変更します。

GitHub の Web サイトから登録する場合は、空のリポジトリを作成した後、次のコマンドで push します。

```sh
git remote add origin https://github.com/<OWNER>/rbi-project.git
git branch -M main
git push -u origin main
```
