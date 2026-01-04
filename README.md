# Docker環境構築（Node.js + MariaDB + Nginx + Redis）

Vagrantを使用してUbuntu 24.04上にDocker環境を自動構築するプロジェクトです。
Ansibleを使用して必要なソフトウェアのインストールと設定を行い、Astro/Node.jsアプリケーションの実行環境を構築します。

## 環境構成

- **OS**: Ubuntu 24.04 (bento/ubuntu-24.04)
- **Docker**: geerlingguy.docker ロールによるインストール
- **タイムゾーン**: Asia/Tokyo

### Dockerコンテナ構成

| コンテナ | 説明 |
|---------|------|
| Node.js | Node.js v24 アプリケーションサーバー |
| Nginx | リバースプロキシ/Webサーバー |
| MariaDB | データベースサーバー |
| Redis | キャッシュサーバー |

## 前提条件

- [VirtualBox](https://www.virtualbox.org/) がインストールされていること
- [Vagrant](https://www.vagrantup.com/) がインストールされていること
- 十分なメモリ（最低2GB）と空きディスク容量があること

## ディレクトリ構成

```
.
├── README.md             # このファイル
├── Vagrantfile           # Vagrant設定ファイル
└── playbooks/            # Ansibleプレイブック
    ├── main.yml              # メインプレイブック
    ├── requirements.yml      # 必要なロールの定義
    ├── vars/                 # 変数定義
    │   └── main.yml              # メイン変数ファイル
    ├── tasks/                # タスク定義
    │   ├── japanese.yml          # 日本語環境設定
    │   ├── redis.yml             # Redisコンテナ構築
    │   ├── mariadb.yml           # MariaDBコンテナ構築
    │   ├── nginx.yml             # Nginxコンテナ構築
    │   ├── nodejs.yml            # Node.jsコンテナ構築
    │   └── app.yml               # アプリケーションデプロイ
    └── containers/           # コンテナ設定
        ├── mariadb/              # MariaDB用Dockerfile等
        ├── nginx/                # Nginx用設定ファイル等
        ├── nodejs/               # Node.js用Dockerfile等
        └── redis/                # Redis用Dockerfile等
```

## IPアドレス設定

仮想マシンには固定IPアドレス `192.168.33.10` が設定されます。
必要に応じて `Vagrantfile` の `config.vm.network` の設定を変更してください。

## 起動手順

1. リポジトリをクローンする
2. 以下のコマンドを実行してVirtualBox上に環境を構築

```bash
vagrant up
```

## 接続方法

環境構築後、以下のいずれかの方法で仮想マシンに接続できます。

1. Vagrantから接続:
```bash
vagrant ssh
```

2. SSHで直接接続:
```bash
ssh vagrant@192.168.33.10
```
※デフォルトパスワード: vagrant

## Dockerコンテナの確認

環境構築後、仮想マシン内で以下のコマンドでコンテナの状態を確認できます。

```bash
docker ps                    # 実行中のコンテナ一覧
docker logs nodejs           # Node.jsコンテナのログ
docker logs nginx            # Nginxコンテナのログ
docker logs mariadb          # MariaDBコンテナのログ
docker logs redis            # Redisコンテナのログ
```

## データベース設定

MariaDBの初期設定は以下の通りです（`playbooks/vars/main.yml` で変更可能）:

| 項目 | 値 |
|------|-----|
| rootパスワード | root_password |
| データベース名 | sample-db |
| ユーザー名 | sample_user |
| パスワード | sample_password |

## カスタマイズ

- `playbooks/vars/main.yml` を編集することで、コンテナ名やDB設定を変更できます
- `playbooks/main.yml` を編集することで、追加のパッケージやタスクを追加できます
- `Vagrantfile` の `vb.memory` を編集して、仮想マシンのメモリ割り当てを変更できます

## アプリケーション

デフォルトでは [next16-mariadb-sample](https://github.com/czbone/next16-mariadb-sample) がデプロイされます。
別のアプリケーションを使用する場合は `playbooks/vars/main.yml` の `app_repo_*` 変数を変更してください。

## トラブルシューティング

### ネットワーク接続の問題
ネットワーク設定に問題が発生した場合は、`Vagrantfile` の IPアドレスを
使用環境に合わせて変更してください。

### 仮想マシンの起動に失敗する場合
VirtualBoxの設定や競合を確認し、必要に応じてVirtualBoxを再起動してください。

### プロビジョニングを再実行する場合
```bash
vagrant provision
```
