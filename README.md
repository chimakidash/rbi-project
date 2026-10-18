# Remote Browser Isolation (RBI) Project

Docker 上の Chromium を Nginx 経由で利用するための構成例です。20ユーザー分のブラウザーサービスを定義しています。

## 特徴

- 15 FPS などの設定により、狭帯域環境での利用を想定しています。実際の帯域や同時利用数は負荷試験で確認してください。
- 共通テンプレートで各ブラウザーコンテナの CPU を 1.0、メモリを 1.2 GB に制限しています。
- Nginx が HTTPS 接続を受け、各ユーザーの Chromium コンテナへ転送します。

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
- 80 番と 443 番ポートが利用可能であること

## Ubuntu への配置と証明書作成

実機では GitHub から clone し、プロジェクトのルートで自己署名証明書を作成します。以下の IP アドレスは利用環境に合わせて変更してください。

```sh
git clone <GitHubリポジトリURL> rbi-project
cd ~/rbi-project
umask 077
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout ./nginx.key -out ./nginx.crt \
  -subj "/CN=10.51.121.10" \
  -addext "subjectAltName=IP:10.51.121.10"
```

このコマンドで秘密鍵 `nginx.key` と証明書 `nginx.crt` を作成します。IP アドレスは利用環境に合わせて変更してください。`nginx.key` と `nginx.crt` は `.gitignore` 対象です。証明書と秘密鍵はサーバー上に保管し、GitHub へ登録しないでください。

## 起動

1. プロジェクトのルートで全コンテナを起動します。

```sh
docker compose config
docker compose up -d
docker compose ps
```

2. ブラウザーから各ユーザーの URL にアクセスします。

- `https://<VMのIPアドレス>/user1/` ～ `https://<VMのIPアドレス>/user20/`

HTTP でアクセスした場合も HTTPS へ転送されます。自己署名証明書のためブラウザーに警告が表示されます。IP アドレスが証明書の SAN と一致することを確認してください。本番運用では、信頼された認証局が発行する証明書を使用してください。

停止する場合:

```sh
docker compose down
```

## 設定について

- Chromium の共通設定で、各サービスに CPU 1.0、メモリ 1.2 GB の上限を指定しています。
- 15 FPS などの帯域設定は構成例であり、実際の画質・帯域・同時利用数は環境と利用内容に応じた負荷試験が必要です。
- Nginx は 443 番で TLS を終端し、80 番へのアクセスを HTTPS に転送します。
- この構成にはユーザー認証がありません。インターネットへ公開せず、利用ネットワークを制限してください。外部公開が必要な場合は、認証・TLS・アクセス制御を先に構成してください。

## 実機での更新

新しい設定を GitHub に push した後、実機で次を実行します。証明書は初回だけ作成し、通常の更新時に再生成しないでください。

```sh
cd ~/rbi-project
git pull --ff-only
docker compose up -d --force-recreate nginx
```

## ログ確認

全コンテナのログをリアルタイムで確認します。終了は `Ctrl+C` です。

```sh
docker compose logs -f
```

個別のログや直近のログは次のコマンドで確認できます。

```sh
docker logs -f rbi-nginx
docker logs -f chromium-user1
docker compose logs | grep -iE 'error|fail'
docker logs --tail 50 chromium-user1
```

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
