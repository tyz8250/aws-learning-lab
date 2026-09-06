# VPC 初回理解メモ

## 状況

VPC について学習中。

CIDR、サブネット、ルーティング、Internet Gateway、Route Table、Security Group などが一気に出てきて混乱していた。

まだルーティングやネットワークの知識はこれから勉強していく段階。

## 最初に整理したかったこと

1. VPC = 大きな自分専用ネットワーク
2. Public Subnet = 外と通信できる区画
3. Internet Gateway = 外への出口
4. Route Table = 外へ出る道の設定
5. Security Group = EC2 の入口ルール
6. EC2(Web) = Public Subnet に置く理由

## 今回わかってきたこと

### VPC

VPC は、AWS 内に作る自分専用のネットワーク空間。

例:

```text
VPC: 10.0.0.0/16
```

これは `10.0.x.x` の範囲を、この VPC の中で使うという意味。

### CIDR

CIDR は IP アドレスの範囲指定。

最初はこの理解でよさそう。

```text
/16 は大きい箱
/24 はその中の部屋
```

例:

```text
10.0.0.0/16    = VPC 全体の大きい範囲
10.0.1.0/24    = Public Subnet の小さい範囲
10.0.101.0/24  = Private Subnet の小さい範囲
```

### Subnet

Subnet は VPC の中を分けた区画。

例:

```text
VPC: 10.0.0.0/16
├── Public Subnet: 10.0.1.0/24
├── Public Subnet: 10.0.2.0/24
├── Private Subnet: 10.0.101.0/24
└── Private Subnet: 10.0.102.0/24
```

Public / Private の違いは、名前だけで決まるわけではない。

大事なのは、Route Table で Internet Gateway に向かう道があるかどうか。

### Internet Gateway

Internet Gateway は VPC とインターネットをつなぐ出口。

ただし、Internet Gateway を VPC に付けただけでは通信できない。

そのサブネットの Route Table に、Internet Gateway へ向かうルートが必要。

### Route Table

Route Table は「通信をどこへ流すか」を決める道案内。

Public Subnet の例:

```text
10.0.0.0/16   local
0.0.0.0/0     Internet Gateway
```

意味:

```text
VPC 内宛ての通信は VPC 内へ
それ以外、つまりインターネット宛ては Internet Gateway へ
```

`0.0.0.0/0` は、すべての宛先という意味。

Private Subnet の例:

```text
10.0.0.0/16   local
0.0.0.0/0     NAT Gateway
```

または、外に出さないなら `local` だけ。

### Security Group

Security Group は EC2 の入口ルール。

Route Table が「道」なら、Security Group は「玄関の鍵」。

Web サーバーの例:

```text
HTTP 80    0.0.0.0/0 から許可
SSH 22     自分の IP だけ許可
```

DB サーバーの例:

```text
DB port    Web サーバーの Security Group からだけ許可
```

### EC2(Web) を Public Subnet に置く理由

ブラウザからアクセスしたい Web サーバーは、インターネットから到達できる必要がある。

そのため Web 用 EC2 は Public Subnet に置く。

一方で DB は外から直接アクセスされたくないので、Private Subnet に置く。

```text
Internet
  ↓
Internet Gateway
  ↓
Public Subnet
  ↓
EC2(Web)
  ↓
Private Subnet
  ↓
EC2(DB)
```

## 今日の重要ポイント

Public Subnet は、名前が Public だから Public なのではない。

```text
0.0.0.0/0 が Internet Gateway に向いているから Public Subnet
```

Private Subnet は、Internet Gateway に直接向かう道がない。

```text
0.0.0.0/0 が Internet Gateway に直接向いていないから Private Subnet
```

## これから勉強すること

- CIDR の読み方
- IP アドレスの範囲計算
- Route Table の読み方
- `local` ルートの意味
- `0.0.0.0/0` の意味
- NAT Gateway の役割
- Public Subnet と Private Subnet の実際の作り方
- Security Group と Network ACL の違い

## 学び

VPC は部品名を暗記するより、通信の流れで見ると理解しやすい。

まずは以下の順番で考える。

1. VPC は大きなネットワーク
2. CIDR は IP 範囲
3. Subnet は VPC を分けた部屋
4. Route Table が Public / Private を決める
5. Internet Gateway は外への出口
6. Security Group は EC2 の入口制限
7. Web は Public、DB は Private に置く

## 次に整理したいこと

実際に VPC を作りながら、以下を確認する。

- VPC の CIDR を決める
- Public Subnet を作る
- Private Subnet を作る
- Internet Gateway を作って VPC に付ける
- Route Table を作る
- Public Subnet に EC2(Web) を置く
- Private Subnet に EC2(DB) を置く想定で構成を考える

## 追記: 実際に作成したもの

### 2026-06-09

VPC に対して Subnet を作成し、Route Table を作成して Subnet に関連付けた。

さらに Internet Gateway を作成し、VPC にアタッチした。

### 操作と理解

#### VPC

自分専用のネットワーク空間を作った。

#### Subnet

VPC の中に小さな区画を作った。

#### Route Table

その区画の通信がどこへ行くかを決める地図を作った。

#### Subnet と Route Table の関連付け

この Subnet ではこの地図を使います、と指定した。

#### Internet Gateway

VPC をインターネットにつなぐ出口を作った。

#### Internet Gateway を VPC にアタッチ

この VPC はインターネットへの出口を持つようになった。

### まだ確認したいこと

- Route Table に `0.0.0.0/0 -> Internet Gateway` を追加したか
- 作成した Subnet が Public Subnet と呼べる状態になっているか
- EC2 を置いたときに、パブリック IP を持たせる必要があるか
- Security Group で HTTP や SSH をどう許可するか
- 作業後に削除すべきリソースは何か
