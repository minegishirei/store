

tttttttttttttt


【参考】https://developer.hashicorp.com/terraform/language/meta-arguments/


## depends_on


depends_on構文は、Terraformが自動的に推測できないリソースやモジュール間の依存関係を明示的に指定するために使用します。
リソースが他のリソースの動作に依存しているが、引数でそのリソースのデータにアクセスしない場合に、depends_onを使用して依存関係を明示します。

- depends_onは最後の手段として使用するべきです。複雑な依存関係はコードの可読性を低下させ、デバッグを困難にします。
- Terraformはリソースの依存関係を自動解析しますが、depends_onを使用すると非効率になる可能性があります。

```tf
resource "aws_iam_role" "example" {
  name = "example"
  assume_role_policy = "..."
}

resource "aws_iam_instance_profile" "example" {
  role = aws_iam_role.example.name
}

resource "aws_iam_role_policy" "example" {
  name   = "example"
  role   = aws_iam_role.example.name
  policy = jsonencode({
    "Statement" = [{
      "Action" = "s3:*",
      "Effect" = "Allow",
    }],
  })
}

resource "aws_instance" "example" {
  ami               = "ami-a1b2c3d4"
  instance_type     = "t2.micro"
  iam_instance_profile = aws_iam_instance_profile.example
  depends_on = [
    aws_iam_role_policy.example
  ]
}
```

上記の例では、`aws_instance`リソースがS3 APIにアクセスするために`aws_iam_role_policy`に依存する必要があるため、depends_onを使用して明示します。




## count


モジュールやリソースに対して複数の同一インスタンスを作成するために使用されます。

```tf
resource "aws_instance" "server" {
    count = 4 # 同じEC2インスタンスを4つ作成
    ami           = "ami-a1b2c3d4"  
    instance_type = "t2.micro"  
    tags = {
        Name = "Server ${count.index}"  
    }
}
```

上記はTerraformを使用して4つのAWS EC2インスタンスを作成するコードです。

countが設定されているブロックでは、${count.index}を使って各インスタンスの構成を変更できます。count.indexは0から始まる個別のインデックス番号です。


同一のインスタンスを複数作成する場合はcountが適切です。
各リソースで異なる設定が必要な場合にはfor_eachを使用します。ただし、可能な限りcountとcount.indexを使用してシンプルに保つことをお勧めします。




## 入力変数


各入力変数はvariableを使用して宣言します。



```tf
variable "image_id" {
  type = string
}

variable "availability_zone_names" {
  type    = list(string)
  default = ["us-west-1a"]
}

variable "docker_ports" {
  type = list(object({
    internal = number
    external = number
    protocol = string
  }))
  default = [
    {
      internal = 8300
      external = 8300
      protocol = "tcp"
    }
  ]
}
```

変数には、以下のオプションを設定できます

- default：デフォルト値
- type：受け入れる値のタイプ
- description：変数の説明
- validation：カスタム検証ルール
- sensitive：機密情報を含む場合に使用
- nullable：nullを許可するかどうか

### 変数の使用

モジュール内で`var`キーワードを使用して変数にアクセスできます。

```tf
resource "aws_instance" "example" {
    instance_type = "t2.micro"
    ami           = var.image_id
}
```


### 変数のオーバーライド

- `-var`オプション

```sh
terraform apply -var="image_id=ami-abc123"
```

- 変数定義ファイル

```sh
terraform apply -var-file="testing.tfvars"
```

- TF_VAR_で始まる環境変数を使用

```sh
$ export TF_VAR_image_id=ami-abc123
$ terraform plan
```





## outputs

Terraformの出力値（outputs）は、インフラストラクチャに関する情報を他のTerraform構成やコマンドラインに公開するための構文です。

- 子モジュールが親モジュールへリソース属性の情報を公開する
- ルートモジュールがCLIで特定の値を表示する
- リモートステートの使用

```tf
output "instance_ip_addr" {
    value = aws_instance.server.private_ip
}
```


親モジュールは、子モジュールの`output`を式で利用できます。例えば、`instance_ip_addr`という出力値は`module.web_server.instance_ip_addr`として参照できます。

- precondition：カスタム条件をチェックし、エラー時にメッセージを表示する。
- description：出力値の目的を説明する。
- sensitive：機密性が高い情報を含むものとしてマークする。


## ローカル変数

ローカル値は、モジュール内で繰り返し使用する値に名前を付けて管理するための機能で、関数の一時的なローカル変数のような役割を果たします。

- ローカル値はlocalsブロック内でまとめて宣言します。

```tf
locals {
    service_name = "forum"
    owner        = "Community Team"
}
```

- 宣言されたローカル値は`local.変数名`で参照します。

```tf
resource "aws_instance" "example" {
    tags = local.common_tags
}
```

注意！ : 変数宣言自体は`locals`ブロックで宣言されますが、参照時には`local`オブジェクトとしてアクセスします。









