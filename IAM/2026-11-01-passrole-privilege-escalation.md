## IAM PassRoleによる権限昇格と最小権限の設計

学習日：2026-11-01
テーマ：AWS IAM / PassRole / AssumeRole / 信頼ポリシー

### 1. 今回の課題

開発者自身には s3:DeleteObject が許可されていない。

しかし、開発者が iam:PassRole を使って、S3の削除権限を持つIAMロールをEC2に渡した結果、EC2上のAWS CLIからS3オブジェクトを削除できた。

なぜ、このようなことが起こるのか？

### 2. 最初の自分の理解

iam:PassRole の Resource: "*" が危険だと考えた。

しかし、当初は Resource: "*" によって、EC2からすべてのリソースに対してあらゆる操作ができると誤解していた。

### 3. 学んだこと

#### PassRoleとAssumeRoleの違い
- iam:PassRole:IAMロールをAWSサービスに渡すことを許可する。
- sts:AssumeRole:IAMロールを引き受け、一時的な認証情報を取得する。
- 信頼ポリシー:誰がIAMロールを引き受けられるかを定義する。
- 許可ポリシー：IAMロールを使用してどの操作を実行できるかを定義する。

開発者自身にS3の削除権限がなくても、強力な権限を持つIAMロールをEC2に渡せる場合、EC2を経由してその権限を利用できる可能性がある。

#### Resourceの意味

Resource: "*"は、指定されたActionの対象リソースを広く指定するものであり、全てのAWS操作を許可する意味ではない。

#### S3の最小原則

EC2からcompany-logsバケットにログをアップロードするだけなら、基本的に次の権限でよい。

```json
{
    "Effect":"Allow",
    "Action":"s3:PutObject",
     "Resource": "arn:aws:s3:::company-logs/*"
}
```
また、s3:ListBucket はバケット自体が対象なので、Resource は arn:aws:s3:::company-logs とする。

### 4.権限昇格を防ぐための設計

- iam:PassRole の Resource を必要なIAMロールのARNに制限する。
- iam:PassedToService で渡す先のサービスを制限する。
- EC2に付与するIAMロールにも最小権限を適用する。
- ほかのポリシーから過剰な許可が与えられていないか確認する。

開発者に直接付与していない権限でも、AWSサービスを経由して間接的に行使できる場合があるため、権限の委譲先まで含めて最小権限を設計する必要がある。

※今回の内容はドリルによる机上学習であり、AWS環境での実機検証は未実施。
