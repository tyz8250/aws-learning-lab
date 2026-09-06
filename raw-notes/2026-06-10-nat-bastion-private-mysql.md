# AWS学習メモ: NAT Gateway・踏み台EC2・Private EC2上のMySQL構築

## 今日やったこと

AWS ハンズオンで、Public Subnet と Private Subnet を使った構成を作成した。

NAT Gateway を作成し、Private Subnet 内の EC2 インスタンスがインターネットへ出ていけるようにした。

また、Public Subnet 側の EC2 を踏み台として利用し、Private Subnet 側の EC2 へ SSH 接続した。

Private EC2 上に MySQL をインストールし、データベースとテーブルを作成して、サンプルデータを保存した。

さらに、Public 側の Web EC2 から Private 側の MySQL へ接続できることを確認した。

## 今回の構成イメージ

```text
Internet
  ↓
Internet Gateway
  ↓
Public Subnet
  ↓
Web EC2（踏み台兼Webサーバー）
  ↓ Private IPで接続
Private Subnet
  ↓
DB EC2 / MySQL
```

Private EC2 は外部から直接アクセスさせず、Public EC2 を経由して接続する構成になっている。

Private EC2 からパッケージ取得などでインターネットへ出る通信は、NAT Gateway を経由する。

## NAT Gatewayとは

NAT Gateway は、Private Subnet 内の EC2 がインターネットへ出ていくための出口。

Private EC2 は外部から直接アクセスされたくないが、パッケージのインストールやアップデートのためにインターネットへ出る必要がある。

今回、Private EC2 で以下を実行したところ、最初はインターネットへ出られず応答が返ってこなかった。

```sh
sudo dnf update -y
```

その後、NAT Gateway を構成することで、Private EC2 から外部リポジトリへアクセスし、MySQL などをインストールできるようになった。

### NAT Gatewayで理解したこと

- NAT Gateway は通常 Public Subnet に置く
- NAT Gateway には Elastic IP が必要
- Private Subnet の Route Table に `0.0.0.0/0 -> NAT Gateway` を設定する
- Private EC2 からインターネットへ出られるが、インターネット側から Private EC2 へ直接接続できるようになるわけではない

## 踏み台EC2とは

踏み台 EC2 とは、Private Subnet 内の EC2 に接続するために経由する Public Subnet 側の EC2 のこと。

Private EC2 はパブリック IP を持たず、外部から直接 SSH できない。

そのため、まず Public EC2 へ SSH 接続し、そこから Private IP を使って Private EC2 へ SSH 接続する。

今回の流れ:

```text
CloudShell またはローカル環境
  ↓
Public EC2へSSH
  ↓
Public EC2からPrivate EC2へSSH
```

## pemファイルをWeb EC2へ送った

Private EC2 へ SSH 接続するため、秘密鍵である `.pem` ファイルを Web EC2 へ送った。

```sh
scp -i udemy-aws-14days.pem \
  udemy-aws-14days.pem \
  ec2-user@<web-ec2-public-ip>:/home/ec2-user
```

その後、Web EC2 から Private EC2 へ SSH 接続した。

```sh
ssh -i udemy-aws-14days.pem ec2-user@<db-ec2-private-ip>
```

### セキュリティ上の重要な注意

今回のハンズオンでは秘密鍵を Web EC2 にコピーしたが、秘密鍵がサーバー上に残るため、実運用では避けたい方法。

今後は以下の方法も学ぶ。

- SSH Agent Forwarding
- SSH ProxyJump
- AWS Systems Manager Session Manager
- 踏み台専用 EC2 と最小権限の Security Group

ハンズオン後は Web EC2 上にコピーした `.pem` ファイルを削除し、秘密鍵の権限も確認する。

## User Dataで初期設定を行った

DB 用 EC2 の起動時に、User Data へ以下を設定した。

```sh
#!/bin/bash

# ホスト名
hostnamectl set-hostname udemy-aws-14days-db-1a

# ロケールの変更
localectl set-locale LANG=ja_JP.UTF-8

# タイムゾーンの変更
timedatectl set-timezone Asia/Tokyo
```

これにより、EC2 起動時にホスト名、ロケール、タイムゾーンを自動設定した。

## User Dataとは

User Data は、EC2 インスタンス起動時に実行できる初期化スクリプト。

毎回手作業で設定するのではなく、インスタンス起動時に必要な初期設定を自動化できる。

今回のように、ホスト名、ロケール、タイムゾーンなどを設定する用途で使える。

### あとで確認したいこと

- User Data の実行ログをどこで確認できるか
- User Data は再起動時にも実行されるのか
- User Data にパスワードや秘密情報を書いてよいのか

## MySQLをPrivate EC2にインストールした

DB 用の Private EC2 で MySQL をインストールした。

```sh
sudo dnf -y install https://dev.mysql.com/get/mysql84-community-release-el9-1.noarch.rpm
sudo dnf -y install mysql mysql-community-server
```

MySQL を起動し、自動起動も有効化した。

```sh
sudo systemctl start mysqld
sudo systemctl enable mysqld
```

root の初期パスワードを確認した。

```sh
sudo more /var/log/mysqld.log | grep 'temporary password'
```

MySQL にログインした。

```sh
mysql -u root -p
```

root ユーザーのパスワードを変更した。

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY '<new-root-password>';
```

実際に使用したパスワードは GitHub に保存しない。

## データベースとテーブルを作成した

MySQL 上で `simple_blog` データベースを作成した。

```sql
CREATE DATABASE simple_blog;
USE simple_blog;
```

投稿データ用の `posts` テーブルを作成した。

```sql
CREATE TABLE posts (
    id INT NOT NULL PRIMARY KEY,
    title VARCHAR(100),
    detail VARCHAR(1000),
    image VARCHAR(1000)
);
```

サンプルデータを登録した。

```sql
INSERT INTO posts VALUES (
    1,
    '[DB] JAWS Days 初参加（2014）',
    '学びが多かった。何より熱量に驚いた。自分も発信する側になりたい。',
    './img/img1.png'
);

INSERT INTO posts VALUES (
    2,
    '[DB] re:Invent 初参加（2016）',
    '規模の大きさに驚いた。個人的には Step Functions の発表が1番よかった。',
    './img/img2.png'
);

INSERT INTO posts VALUES (
    3,
    '[DB] AWS 設計 に関する本を執筆しました（2018）',
    '多くの方に読んでいただけたら嬉しいです。',
    './img/img3.png'
);

INSERT INTO posts VALUES (
    4,
    '[DB] AWS SAA 資格対策の本を執筆しました（2019）',
    'オリジナル問題を通して対策していただけます。',
    './img/img4.png'
);
```

## アプリ用MySQLユーザーを作成した

Web EC2 から MySQL へ接続するため、アプリ用のユーザーを作成した。

```sql
CREATE USER 'simple_blog_user'@'%' IDENTIFIED BY '<app-user-password>';
GRANT ALL PRIVILEGES ON simple_blog.* TO 'simple_blog_user'@'%';
```

これにより、`simple_blog_user` が `simple_blog` データベースへアクセスできるようになった。

### セキュリティ上の注意

- 実際のパスワードは GitHub に保存しない
- `root` ユーザーをアプリ接続に使わない
- 接続元を `%` にすると広い範囲から接続可能になるため、Security Group とユーザーの接続元を必要最小限にする
- アプリユーザーに `WITH GRANT OPTION` を付ける必要性を確認する
- 必要な操作だけに権限を絞ることを今後学ぶ

## Web EC2からMySQLへ接続した

Web 用 EC2 側にも MySQL クライアントと PHP 用の MySQL 拡張をインストールした。

```sh
sudo dnf -y install https://dev.mysql.com/get/mysql84-community-release-el9-1.noarch.rpm
sudo dnf -y install mysql php-mysqlnd
```

Web EC2 から Private EC2 上の MySQL へ接続した。

```sh
mysql -h <db-ec2-private-ip> -u simple_blog_user -p
```

これにより、Public Subnet 側の Web EC2 から、Private Subnet 側の DB EC2 へ接続できることを確認した。

### Security Groupで確認したいこと

- DB EC2 の MySQL 3306 番ポートをインターネット全体へ開放しない
- DB EC2 の 3306 番ポートは、Web EC2 の Security Group からだけ許可する
- DB EC2 の SSH 22 番ポートも、踏み台 EC2 の Security Group からだけ許可する

## PHPファイルを配置した

講師のリポジトリ内にある `index.php` を Apache の公開ディレクトリへコピーした。

```sh
sudo cp udemy-aws-14days/Day04/index.php /var/www/html/
```

配置した PHP ファイルの内容も確認した。

```sh
sudo cat /var/www/html/index.php
```

### セキュリティ上の注意

`index.php` 内に DB のパスワードを直接記述している場合は、GitHub や画面出力に含めない。

今後は環境変数や AWS Secrets Manager、AWS Systems Manager Parameter Store などで認証情報を管理する方法も学ぶ。

## 疎通確認

Web EC2 に対して HTTP アクセスし、フロント画面が表示されることを確認した。

```sh
curl http://<web-ec2-public-ip>/
```

一方で、DB EC2 は Private Subnet 側にあり、外部から直接アクセスさせない構成にしている。

## 今日の構成で重要なポイント

今回の構成では、Web サーバーと DB サーバーの役割を分けた。

Web EC2 は Public Subnet に配置し、外部から HTTP アクセスできるようにした。

DB EC2 は Private Subnet に配置し、外部から直接アクセスできないようにした。

ただし、DB EC2 からインターネットへ出る必要があるため、NAT Gateway を使って外部リポジトリへアクセスできるようにした。

```text
インターネットからWebへの通信:
Internet -> Internet Gateway -> Public Subnet -> Web EC2

WebからDBへの通信:
Web EC2 -> Private IP -> Private Subnet -> DB EC2:3306

Private DBから外部への通信:
DB EC2 -> Private Route Table -> NAT Gateway -> Internet Gateway -> Internet
```

## 今日の学び

Public Subnet と Private Subnet の違いが、実際の構成を通して少し見えてきた。

Public Subnet は、外部からアクセスされる Web サーバーを置く場所。

Private Subnet は、外部から直接アクセスされたくない DB サーバーを置く場所。

Private Subnet の EC2 は外部から直接 SSH できないため、Public EC2 を踏み台として接続する。

また、Private EC2 がパッケージインストールなどでインターネットへ出るには、NAT Gateway が必要になる。

今回、Private EC2 上に MySQL を構築し、Web EC2 から Private IP で MySQL へ接続できることを確認した。

これにより、Web サーバーと DB サーバーを分離した基本的な AWS 構成を体験できた。

## 削除・課金対策

NAT Gateway は起動時間と処理データ量に応じて料金が発生するため、学習後は特に削除を忘れない。

確認対象:

- NAT Gateway
- NAT Gateway 用 Elastic IP
- Web EC2
- DB EC2
- EBS ボリューム
- Internet Gateway
- Route Table
- Public Subnet
- Private Subnet
- Security Group
- VPC

削除前に、リソース間の依存関係と削除順序を確認する。

## 次に整理したいこと

- NAT Gateway を置く Subnet と Route Table の関係
- 踏み台 EC2 と Session Manager の違い
- Security Group を Security Group ID で参照する方法
- MySQL ユーザーの権限を最小限にする方法
- PHP から DB 認証情報を安全に読み込む方法
- NAT Gateway の料金と、学習環境での代替手段
- 今回の構成を Mermaid または draw.io で図にする
