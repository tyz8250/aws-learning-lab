# AWS学習メモ: EC2上でApache・Git・PHPアプリを動かした

## 今日やったこと

EC2 インスタンス上に Apache をインストールした。

さらに Git もインストールし、講師の GitHub リポジトリを `git clone` で取得した。

その後、クローンしたリポジトリ内の PHP ファイルを使ってサーバーを起動し、ブラウザからフロント画面を表示できることを確認した。

## 今回の流れ

```text
EC2 へ SSH 接続
↓
Apache をインストール
↓
Git をインストール
↓
講師の GitHub リポジトリを git clone
↓
PHP ファイルを使ってサーバーを起動
↓
ブラウザからアクセス
↓
フロント画面が表示された
```

## Apache とは

Apache は Web サーバーソフトウェアのひとつ。

EC2 上に Apache をインストールすることで、EC2 を Web サーバーとして利用できる。

ブラウザから HTTP でアクセスしたときに、EC2 上のファイルやアプリケーションを表示する役割を持つ。

## Git をインストールした理由

Git は、ソースコードの管理や取得に使うツール。

今回は、講師が用意した GitHub リポジトリを EC2 上に取得するために Git をインストールした。

使用した操作は `git clone`。

```sh
git clone <repository-url>
```

これにより、GitHub 上のコードを EC2 インスタンス内にコピーできる。

## PHP ファイルを使ったサーバー起動

クローンしたリポジトリ内の PHP ファイルを使い、サーバーを起動した。

その結果、ブラウザから EC2 のパブリック IP アドレスなどにアクセスし、フロント画面が表示されることを確認できた。

## 今回難しかったこと

AWS だけでなく、Linux コマンド、Apache、Git、PHP、ネットワーク設定が同時に出てきたため、かなり難しく感じた。

特に、どのディレクトリにいるのか、どのコマンドをどこで実行するのか、ファイルがどこにあるのかを把握するのが難しかった。

## 今日の学び

EC2 は単なる仮想サーバーではなく、その上に Web サーバーやアプリケーションを配置して動かすことができる。

Apache を入れることで HTTP アクセスを受けられるようになり、Git を使うことで GitHub 上のコードを EC2 へ取得できる。

今回、EC2 上にアプリケーションを配置し、PHP ファイルを使ってサーバーを起動し、ブラウザでフロント画面を表示するところまで確認できた。

この作業を通して、AWS、Linux、Web サーバー、Git、アプリケーション起動の流れがつながり始めた。

## あとで整理したいこと

- Apache で表示しているものと、PHP の組み込みサーバーで表示しているものの違い
- 実際に使った PHP サーバー起動コマンド
- どのディレクトリで `git clone` したか
- クローンしたリポジトリをどこに置いたか
- Security Group で開けたポート
- ブラウザからアクセスした URL の形式
- 作業後に EC2、EBS、Elastic IP などを削除したか

## セキュリティ上の注意

- EC2 の Public IP は必要がなければ公開しない
- `.pem` 秘密鍵は GitHub に載せない
- SSH 22 番は自分の IP に絞る
- HTTP 80 番や PHP の検証用ポートを開けた場合は、学習後に閉じる
- 講師のリポジトリ URL を公開してよいか確認する

## 削除・課金対策

- 学習後に EC2 を停止または終了する
- EBS ボリュームが残っていないか確認する
- Elastic IP を使った場合は解放する
- 不要な Security Group の公開ルールを削除する
- クローンしたコードに秘密情報を書き込んでいないか確認する

## 今日のメモ

EC2 上で Apache、Git、PHP アプリがつながり、ブラウザからフロント画面を確認できた。

## 追記: PHP 8.4、Apache 設定、アプリ配置

### 2026-06-09

PHP 8.4 をインストールし、Apache の設定を変更した。

その後、Git をインストールして講師リポジトリを clone し、`Day03` のファイルを Apache の公開ディレクトリにコピーした。

### PHP 8.4 のインストール

```sh
sudo dnf install -y php8.4
```

理解:

- PHP アプリを動かすために PHP 8.4 をインストールした
- Apache で PHP ファイルを扱う準備をした

### Apache 設定ファイルの編集

```sh
sudo vim /etc/httpd/conf/httpd.conf
```

変更内容:

```apache
<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>
```

理解:

- ディレクトリにアクセスされたとき、`index.php` を優先して表示するようにした
- `index.html` より先に `index.php` を探す設定にした

追加または変更した設定:

```apache
ServerName udemy-aws-14days-web-1a
```

理解:

- Apache のサーバー名を指定した
- 起動時や設定確認時の警告を減らす目的がある

### Apache 設定の文法確認

```sh
httpd -t
```

理解:

- Apache の設定ファイルに文法エラーがないか確認した
- 設定変更後、すぐ reload する前に確認するのが安全

### Apache の reload

```sh
sudo systemctl reload httpd
```

理解:

- Apache を完全停止せず、設定を再読み込みした
- `httpd.conf` の変更を反映した

### Git のインストール

```sh
sudo dnf install -y git
```

理解:

- GitHub 上のコードを EC2 に取得するために Git をインストールした

### 講師リポジトリの clone

```sh
git clone https://github.com/ketancho/udemy-aws-14days.git
```

理解:

- 講師が用意したサンプルコードを EC2 上にコピーした
- 公開リポジトリなので URL はこのメモに残してよい

### Day03 のファイルを Apache 公開ディレクトリへコピー

```sh
sudo cp -r udemy-aws-14days/Day03/* /var/www/html/
```

理解:

- `Day03` 配下のファイルを Apache が配信する `/var/www/html/` に配置した
- ブラウザから EC2 にアクセスしたとき、このディレクトリのファイルが表示対象になる

### まだ確認したいこと

- `/var/www/html/` にどのファイルがコピーされたか
- `index.php` が存在しているか
- Apache が PHP を正しく実行できているか
- ブラウザ表示時に PHP のコードがそのまま表示されていないか
- `httpd -t` の結果が `Syntax OK` だったか
- Security Group で HTTP 80 番を許可しているか
- 作業後に EC2 を停止または終了したか
