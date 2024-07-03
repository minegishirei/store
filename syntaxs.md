Terraformのステート管理

tttttttttttttttttttttttt



Terraformはapplyを実行するたびに、以前作成したリソースを見つけて更新することができます。
この時、Terraformは**State管理**と呼ばれる方法でクラウドに存在する自身が管理しているリソースを見つけています。


- Terraformステートとは
- ステートファイルを共有ストレージで管理する（Terraformバックエンド）
- ステートファイルの分離方法2種
- `terraform_remote_state`

Terraformを実行する場合、Terraformは構築したリソースの情報を **ステートファイル**に格納します。
デフォルトでは`terraform apply`実行時のカレントディレクトリに ステートファイルである`.tfstate`というファイルを作成し、そこで情報を管理しています。

`.tfstate` は以下のようなjson形式のファイルです。
`id`属性等を使用してAWSリソースの中から特定の情報を見つけ出します。

```json
{
    "version": 1,
    "serial": 7,
    "modules": [
        {
            "path": [
                "root"
            ],
            "outputs": {},
            "resources": {
                "aws_route53_record.site": {
                    "type": "aws_route53_record",
                    "depends_on": [
                        "aws_route53_zone.primary",
                        "aws_s3_bucket.site",
                        "aws_s3_bucket.site"
                    ],
                    "primary": {
                        "id": "Z2OIQETM3FU6D_mikeball.me_A",
                        "attributes": {
                            "alias.#": "1",
                            "alias.701075612.evaluate_target_health": "false",
                            "alias.701075612.name": "s3-website-us-west-2.amazonaws.com",
                            "alias.701075612.zone_id": "Z3BJ6K6RIION7M",
                            "failover": "",
                            "fqdn": "mikeball.me",
                            "health_check_id": "",
                            "id": "Z2OIQETM3FU6D_mikeball.me_A",
                            "name": "mikeball.me",
                            "records.#": "0",
                            "set_identifier": "",
                            "ttl": "0",
                            "type": "A",
                            "weight": "-1",
                            "zone_id": "Z2OIQETM3FU6D"
                        }
                    }
                },
                ...
```


## Gitでステートファイルを管理するべきではない理由

仮に個人でTerraformを使用しているのであれば、ローカルに置いたままでも特に問題はありません。
しかし、実際のプロダクトでチーム開発を行う際には、 **ローカルでステートファイルを操作することで個人間で矛盾が生じする恐れがあります**

また、このステートファイルは **AWSリソースの管理状態という実態を持つもの**であるため、gitで管理する方法も以下の理由で避けた方が良いです。

- ステートファイルのプルが漏れたり、プッシュが漏れたりすることで、 **どのステートファイルが最新のものかがわからなくなってしまう**
- ステートファイル内部には機密情報が記載されることがある (AWS Auroraのアドミンユーザーネームやパスワードなど) これはソースコードと一緒にバージョン管理するべきではない。

Terraformのソースコード自体はGitで管理することは問題ありませんが、StateファイルをGitで管理すると個人間で矛盾が生じる恐れがあります。


## チームでステートファイルを管理する

そこで、 Terraformのソースコードを管理するためには **リモートバックエンド** と呼ばれる仕組みを使うことをおすすめします。
リモートバックエンドはTerraformが提供してくれるステートファイルの管理方法で、以下の特徴があります。

- リモートバックエンドは `apply`,`plan`コマンドの実行のたびに自動的にステートファイルをロードしてくれますし、
`apply`コマンド実行のたびに新しく更新されたステートファイルをアップロードしてくれます。
そのため、コミット漏れによる個人間のステートの矛盾は起こりません。
- また、リモートバックエンドは暗号化をサポートしているため、S3に保存されたステートファイルからパスワードを確認されることもありません。

TerraformをAWSで使用する場合は、S3をリモートバックエンドとして使うのが良いです。
S3だと上記の特徴に加えて、以下のメリットがあります。

- S3はバージョニングをサポートしてくれるので、昔のバージョンに戻りたいときはロールバックが可能です。
- 安価です。
- ほぼ(99.99%)壊れません。
- 言ってしまえばただのファイル置き場なので、シンプルです。

Terraformでリモートバックエンドを指定するには、Terraformソースコード内に`backend`構文を使用します。

```py
terraform {
    backend "s3" {
        bucket          = "terraform-state-<your_project_name>"
        key             = "global/s3/terraform.tfstate"
        region          = "ap-northeast-1"
        encrypt         = true
    }
}
```

このコードを配置した後に、`terraform init`を実行すると、Terraformはステートファイルがローカルに存在することを検知して
それをs3リモートバックエンドに保存するかどうか尋ねられます。 `yes`と答えると、指定したバケットとキーにステートファイルが保存されます。
























