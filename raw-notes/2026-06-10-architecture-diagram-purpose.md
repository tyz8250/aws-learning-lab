# AWS構成図を書く目的

## 今日のメモ

Udemy 講師より、ホワイトボードなどに AWS 構成図を書いてみるとよいとの説明があった。

AWS は GUI でリソースを作成できるため、操作自体は進めやすい。

一方で、VPC、Subnet、Route Table、Internet Gateway、EC2 などの関係を理解するには、自分で構成図を書けることが重要だと感じた。

## 構成図を書くことで整理できること

- どのリージョンに作っているか
- どの VPC の中に Subnet があるか
- Public Subnet と Private Subnet の違い
- Route Table がどの Subnet に関連付いているか
- Internet Gateway へのルートがあるか
- EC2 がどの Subnet に配置されているか
- インターネットから EC2 まで通信がどう流れるか

## Public Subnet についての理解

特に Public Subnet は、Subnet を作るだけでは Public Subnet として機能するわけではない。

Route Table に以下のルートがあることで、Public Subnet として機能する。

```text
0.0.0.0/0 -> Internet Gateway
```

つまり、Public Subnet かどうかは名前ではなく、外へのルートがあるかどうかで決まる。

## 今後の方針

今後は AWS のハンズオンで作った内容を、単に画面操作で終わらせず、簡単な構成図として書き出すようにする。

構成図を書くことで、作ったリソース同士の関係や通信の流れを自分の言葉で説明できるようにする。

## あとで整理したいこと

- VPC の中に Public Subnet / Private Subnet をどう描くか
- Route Table と Subnet の関連付けを図でどう表すか
- Internet Gateway へのルートを図でどう見せるか
- EC2(Web) を Public Subnet に置く理由を図で説明する
- EC2(DB) を Private Subnet に置く理由を図で説明する
- Mermaid または draw.io で構成図を残す
