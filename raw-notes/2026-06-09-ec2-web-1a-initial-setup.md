# EC2 Web-1a 初期設定メモ

## 状況

VPC 上に EC2 インスタンスを作成し、SSH 接続、ホスト名変更、ロケール変更、タイムゾーン変更、dnf 更新、Apache のインストールと起動を行った。

ターミナル上でも日本語表記になり、使いやすいと感じた。

## 実行したこと

### SSH 接続

```sh
ssh -i udemy-aws.pem ec2-user@<Web-1a instance public ip>
```

メモ:

- `<Web-1a instance public ip>` は GitHub にそのまま載せない
- 秘密鍵 `.pem` の中身は絶対に載せない
- 後続のコマンドでは `udemy-aws-14days.pem` になっているため、実際に使った鍵ファイル名をあとで確認する

### ホスト名の変更

```sh
sudo hostnamectl set-hostname udemy-aws-14days-web-1a
```

理解:

- EC2 インスタンスの OS 上のホスト名を変更した
- 複数台構成になったときに、どのサーバーに入っているか分かりやすくなる

### 再 SSH してホスト名確認

```sh
exit
ssh -i udemy-aws-14days.pem ec2-user@<Web-1a instance public ip>
```

理解:

- 一度ログアウトして再接続することで、ターミナル表示上のホスト名変更を確認した
- 鍵ファイル名が最初の SSH コマンドと異なるため、正しいファイル名をあとで整理する

### ロケールの変更

```sh
localectl status
sudo localectl set-locale LANG=ja_JP.UTF-8
source /etc/locale.conf
localectl status
```

理解:

- OS の表示言語や文字コード設定を日本語 UTF-8 に変更した
- 日本語表示できると、学習中にコマンド結果を読みやすい
- `source /etc/locale.conf` で現在のシェルに設定を反映した

### タイムゾーンの変更

```sh
timedatectl
sudo timedatectl set-timezone Asia/Tokyo
timedatectl
```

理解:

- サーバーのタイムゾーンを日本時間に変更した
- ログ確認や作業時刻の把握がしやすくなる
- CloudWatch や AWS コンソール側の時刻表示とは違う場合があるので注意する

### dnf パッケージ更新

```sh
sudo dnf update -y
```

理解:

- インストール済みパッケージを最新状態に更新した
- セキュリティ更新や不具合修正を取り込むための基本作業

### Apache のインストールと起動

```sh
sudo dnf install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd
```

理解:

- `httpd` は Apache Web サーバーのパッケージ
- `systemctl start httpd` で Apache を起動した
- `systemctl enable httpd` で EC2 再起動後も自動起動するようにした

## まだ確認したいこと

- Apache が起動しているか
- EC2 の Security Group で HTTP 80 番を許可しているか
- ブラウザから `http://<public-ip>` にアクセスできるか
- Public Subnet の Route Table に `0.0.0.0/0 -> Internet Gateway` があるか
- EC2 にパブリック IPv4 が割り当てられているか
- 鍵ファイル名は `udemy-aws.pem` と `udemy-aws-14days.pem` のどちらが正しいか

## セキュリティ上の注意

- `.pem` ファイルは GitHub に載せない
- Public IP を公開する必要がなければ伏せる
- SSH 22 番は自分の IP だけに制限する
- HTTP 80 番を開ける場合は、学習用途であることを意識する
- Apache を起動したまま放置しない

## 削除・課金対策

- 学習後に EC2 を停止または終了する
- EBS ボリュームが残っていないか確認する
- Elastic IP を使った場合は解放する
- AMI やスナップショットを作った場合は削除する
- Security Group の不要な公開ルールを消す

## 学び

VPC 上に EC2 を作ったあと、OS 側の初期設定としてホスト名、ロケール、タイムゾーン、パッケージ更新、Apache 起動を行った。

ネットワーク側では、Public Subnet、Route Table、Internet Gateway、Security Group がそろっていないと、Apache を起動してもブラウザからアクセスできない。
