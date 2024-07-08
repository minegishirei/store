

ttttttttttttttttttttt



Terraformにおける環境の分離は、いくつか手段があります。

- ワークスペースの分離
- ファイルレイアウトによる分離


ワークスペースによる分離は環境間の完全な分離を保証できません。

- バックエンドが同じリソースを使用してしまう（AWSであれば同じS3バケットに保存されてしまう）
- `terraform workspace`コマンドを実行しない限り、どのワークスペースを使用中なのかが確認できない

これにより、ワークスペースの使用は間違いを引き起こしがちになり、ともすれば間違ったワークスペースにデプロイしてしまう可能性もあります。
これらの欠点から、ステージング環境と本番環境の分離にワークスペースを活用するのは適切ではありません。

主観 : **もし完全な分離を望むのであれば、AWSアカウントごとの分離がベストですが、各企業のアカウント払出し事情に応じてできない可能性もあります。**
その場合、環境の分離にはファイルごと分離することがおすすめです。


## ファイルレイアウトによる分離

各環境の完全な分離のためには、次の手段が考えられます。

- 各環境用のTerraformファイルをそれぞれ別のフォルダに入れます。
    - `stage` : ステージング環境(動作確認を行う環境)
    - `prod`  : 本番環境
    - `mgmt`  : DevOpsツール用の環境
- 環境ごとに異なるバックエンド(AWSであればことなるS3バケット)を使用します。









## モジュールを使わない場合の構成

Terraformで複数の環境を用意する場合、モジュールを使用しない場合は、コピーアンドペーストで重複の多いコードになりますが、
主観的にも経験則的にもそれで問題ないです。

まずフォルダ構成のベストプラクティスとして、
各環境用のTerraformファイルをそれぞれ別のフォルダに入れます。

- `stage` : ステージング環境(動作確認を行う環境)
- `prod`  : 本番環境
- `mgmt`  : DevOpsツール用の環境
- `global` : 上記の環境で共通して使うリソース

さらに各環境の構成図は以下の通りです。

```
├── global
│   └── s3
│       ├── README.md
│       ├── main.tf
│       ├── outputs.tf
│       └── variables.tf
│── stage
│   ├── data-stores
│   │   └── mysql
│   │       ├── README.md
│   │       ├── main.tf
│   │       ├── outputs.tf
│   │       └── variables.tf
│   └── services
│       └── webserver-cluster
│           ├── README.md
│           ├── main.tf
│           ├── outputs.tf
│           ├── user-data.sh
│           └── variables.tf
│── prod
│   ├── data-stores
│   │   └── mysql
│   │       ├── README.md
│   │       ├── main.tf
│   │       ├── outputs.tf
│   │       └── variables.tf
│   └── services
│       └── webserver-cluster
│           ├── README.md
│           ├── main.tf
│           ├── outputs.tf
│           ├── user-data.sh
│           └── variables.tf
```

各環境のフォルダは以下には、コンポーネントごとのフォルダがあります。
コンポーネントもプロジェクトごとに異なりますが、よく使われるのは以下の通りです。

- `vpc` : 各環境のネットワークトポロジー
- `services` : この環境で動かすアプリケーションがあるフォルダー。RubyやPythonなどの各言語のビジネスロジックはここに集約される。
マイクロサービスの場合はさらに細かく分離される。
- `data-storage` : 永続化層のこと。この環境で使うMySQLやAWS Aurora、DynamoDBなどのデータストア。

これらの配下には次のようなファイルが置かれます。
特に、Terraformでは**入力変数と出力変数を明記することで、その他の開発者がコードの意図を推測しやすくなります。**

- 必須レベルのファイル構成
    - `variables.tf` : 入力変数
    - `outputs.tf` : 出力変数
    - `main.tf` : その他メインで使用するリソース
    - `providers.tf` : 使用するプロバイダとどんな認証情報が必要になるかがわかります。
- おすすめのファイル構成
    - `dependencies.tf` : データソースをすべてdependencies.tfに入れることで、コードが外部に依存している部分をわかりやすくすることもできます。
    - `main-xxx.tf` : リソースが増えて`main.tf`が長くなり出したらさらに分割しても良いです。
        - 分け方はさまざまであり、 `main-s3.tf`や `main-iam.tf`などリソース単位の分割もありえます。
        - また、ec2やecsなどのサーバーは `main-compute.tf`に分けるなど役割単位での分割もあります。

**モジュールを使おうと、使わなかろうと、 これらのファイル構成を意識しておくことは推測しやすいコードを書くために重要です。**



## モジュールを使う場合の構成

通常は社内のステージング環境と、実際のユーザーがアクセスする本番環境の少なくとも二つが必要になります。
Terraformで複数の環境を用意する場合、モジュールを使用しない場合は、コピーアンドペーストで重複の多いコードになります。

主観的にも経験則的にもそれで問題ないですが、再利用性をさらに向上させるためには、IaCコードをモジュール化する必要があります。

**Terraformモジュールを使うことで、同じコードをコピペを使わずステージング環境と本番環境間で使いまわせます。**

モジュールを使う時のよくある構成図は以下の通りです。
`prod`と`stage`以外に、 `modules`という共通部品フォルダーがあるのが確認できます。

```
├── modules
│   └── services
│       └── webserver-cluster
│           ├── README.md
│           ├── main.tf
│           ├── outputs.tf
│           ├── user-data.sh
│           └── variables.tf
├── prod
│   ├── data-stores
│   │   └── mysql
│   │       ├── README.md
│   │       ├── main.tf
│   │       ├── outputs.tf
│   │       └── variables.tf
│   └── services
│       └── webserver-cluster
│           ├── README.md
│           ├── main.tf
│           ├── outputs.tf
│           └── variables.tf
└── stage
    ├── data-stores
    │   └── mysql
    │       ├── README.md
    │       ├── main.tf
    │       ├── outputs.tf
    │       └── variables.tf
    └── services
        └── webserver-cluster
            ├── README.md
            ├── main.tf
            ├── outputs.tf
            └── variables.t
```



### モジュールの基本

Terraformモジュールはとてもシンプルな構造であり、**あるフォルダ内にあるTerraform設定ファイルの集まりは、モジュールになります。**
ただし、フォルダ内部では `provider`定義を削除しておきましょう。
これはモジュールを呼び出す側で定義するものです。

プロバイダー定義を削除したらモジュールを使用できます。
モジュールを使用するためには次のように文法を宣言します。(本番環境想定)

```js
module "webserver_cluster" {
  source = "../../../modules/services/webserver-cluster"

  cluster_name           = var.cluster_name
  db_remote_state_bucket = var.db_remote_state_bucket
  db_remote_state_key    = var.db_remote_state_key

  instance_type = "m4.large"
  min_size      = 2
  max_size      = 10
}

provider "aws" { # providerはモジュールの呼び出しがわで宣言する。
  region = "us-east-2"
}
```

- `source` でモジュールフォルダーを指定します。
- それ以下に続く引数の設定は、モジュール内部で宣言した `variables`に格納される値です。
**モジュールに対する入力引数の設定は、リソースに対する引数の設定と同じ文法を使えます。**
- このソースコードは `prod`フォルダー配下で管理することをお勧めします。

上記は本番環境の設定なので、 EC2のスケールは2~10、インスタンスタイプは`m4.large`で、少しリッチな性能をしています。
ステージング環境を作成するときは、本番環境と同程度のスペックは必要ありません。
以下はステージング環境の要件に合わせてスペックを落としてます。

```js
module "webserver_cluster" {
  source = "../../../modules/services/webserver-cluster"

  cluster_name           = var.cluster_name
  db_remote_state_bucket = var.db_remote_state_bucket
  db_remote_state_key    = var.db_remote_state_key

  instance_type = "t2.micro"
  min_size      = 2
  max_size      = 2
}

provider "aws" { # providerはモジュールの呼び出しがわで宣言する。
  region = "us-east-2"
}
```

上記はEC2インスタンスのスケールを2で固定、インスタンスタイプは `t2.micro` でスペックを落としてます。
**これでステージング環境のコストを抑えつつ、本番環境で十分な要件のスペックを担保し、それぞれのコードでコアとなる共有部分はまとめて管理することができました。**

### モジュールの注意点



ただし、モジュールを使用する際には二点注意する必要があります。

- 1.terraformで管理されているリソース名がハードコーディングされていないことに気をつけてください。
AWSでは同じアカウント内で同一の名前を使うことができないため、ステージング環境と本番環境で同じアカウントで運用する場合は重複しないように気をつけましょう。
**モジュールの中のリソース名はハードコーディングせず、`variables`をうまく使用して名前の重複を避けましょう。**

```js
resource "aws_autoscaling_group" "example" {
  launch_configuration = aws_launch_configuration.example.name
  vpc_zone_identifier  = data.aws_subnets.default.ids
  target_group_arns    = [aws_lb_target_group.asg.arn]
  health_check_type    = "ELB"

  min_size = var.min_size
  max_size = var.max_size

  tag {
    key                 = "Name"
    value               = "${var.cluster_name}-asg"
    propagate_at_launch = true
  }
}
```

上記の例では `Name`タグに `"${var.cluster_name}-asg"`を当てることで `クラスタ名 + asg`という名前が充てられることが保証できます。


- 2.`terraform_remote_state` データソースを使っている場合、ステージング環境も本番環境も同じソースコードを参照してしまっているので、
データベースステートの読み取り方も変数を使用してハードコードを回避する必要があります。



### 入力値のモジュール側の設定

moduleを呼び出す際に、引数を設定して環境ごとに設定を変更する場合、
引数は `variables.tf` でまとめて宣言することをお勧めします。



```js
# ---------------------------------------------------------------------------------------------------------------------
# REQUIRED PARAMETERS
# You must provide a value for each of these parameters.
# ---------------------------------------------------------------------------------------------------------------------

variable "cluster_name" {
  description = "The name to use for all the cluster resources"
  type        = string
}

variable "db_remote_state_bucket" {
  description = "The name of the S3 bucket for the database's remote state"
  type        = string
}

variable "db_remote_state_key" {
  description = "The path for the database's remote state in S3"
  type        = string
}

variable "instance_type" {
  description = "The type of EC2 Instances to run (e.g. t2.micro)"
  type        = string
}

variable "min_size" {
  description = "The minimum number of EC2 Instances in the ASG"
  type        = number
}

variable "max_size" {
  description = "The maximum number of EC2 Instances in the ASG"
  type        = number
}

# ---------------------------------------------------------------------------------------------------------------------
# OPTIONAL PARAMETERS
# These parameters have reasonable defaults.
# ---------------------------------------------------------------------------------------------------------------------

variable "server_port" {
  description = "The port the server will use for HTTP requests"
  type        = number
  default     = 8080
}
```







### モジュールの値を取得する

Terraformモジュールは `outputs`を使用することで値を返すことができます。
これはプログラミングでの関数の戻り値に該当する機能です。

例えば、AWSリソースでEC2のオートスケーリンググループを使用したサーバー群を立ち上げたとき、次のように返却値を設置すればテストなどが容易になります。
以下の出力値のソースコードは `outputs.tf`でまとめて宣言することを推奨します。

```js
output "alb_dns_name" {
  value       = aws_lb.example.dns_name
  description = "The domain name of the load balancer"
}

output "asg_name" {
  value       = aws_autoscaling_group.example.name
  description = "The name of the Auto Scaling Group"
}

output "alb_security_group_id" {
  value       = aws_security_group.alb.id
  description = "The ID of the Security Group attached to the load balancer"
}
```



## モジュールのバージョン管理

ステージング環境と本番環境の両環境が同じモジュールのフォルダを指定しているときに、そのモジュールフォルダに変更を加えた場合は
両環境で変更が発生してしまいます。
その場合、**ステージング環境と本番環境で同じモジュールの別々のバージョンを使用することで、両環境を疎結合に保てます。**

今までモジュールを呼び出すときには、`source`パラメーターにモジュールのファイルパスを指定してました。
しかし、**Terraformはファイルパスに加えてGitのURLなどのリモートのソース管理システムを指定することができます。**

`modules` : このリポジトリでは、再利用可能なモジュールを定義します。



















