VPC、サブネット、セキュリティグループの違い
AWS
tttttttttttttttttttt






# VPCとは何か?

VPCとは**仮想ネットワーク**です。
ひとつのAWSアカウントを論理的に分離することができる仮想ネットワークで AWS リソースを起動できるようになります。

VPCの全体像は以下の通りです。

<img src="https://github.com/minegishirei/store/blob/main/aws/network/subnet-diagram.png?raw=true">

VPCはさらに細かい **サブネット** という小さいネットワークに分割することができます。
サブネットにはパブリックサブネットとプライベートサブネットの2種類があります。これらの領域に分割することで、セキュリティを担保できるのです。

- パブリックサブネット: 直接インターネットアクセスが可能なサブネット。
- プライベートサブネット: インターネットアクセスが制限されているサブネット。



## VPCのIaCコードサンプル

```yml
ProductVPC:
  Type: AWS::EC2::VPC
  DeletionPolicy: Delete        # Deleteは、スタックが削除されるときにVPCも削除することを意味します。
  Properties:
    CidrBlock: 172.16.0.0/16    # VPCのCIDRブロックを指定します。CIDRブロックはVPC内のIPアドレスの範囲を定義します。
    EnableDnsSupport: 'true'
    EnableDnsHostnames: 'true'
    InstanceTenancy: default    # VPC に起動されたインスタンスは、インスタンスの起動時に別のテナンシーを明示的に指定しない限り、デフォルトで共有ハードウェア上で実行されます。基本はデフォルトを指定する。
    Tags:
    - Key: Name
      Value: ProductVPC
```

- `CidrBlock: 172.16.0.0/16`
    - CIDRブロックはVPC内部で取れるIPアドレスの範囲です。
    - 上記のCIDRブロックを次のサイトに入力すると、IPアドレスのとりうる範囲がわかります。
    - https://www.rohde-schwarz.com/jp/solutions/cybersecurity/landing-pages/cidr-calculator_256249.html
    - `172.16.0.0 ... 172.16.255.255`
    - Amazon VPCに割り当てるIPv4のCIDRブロックですが、AWSではRFC 1918 で定義されているプライベートIPアドレスの範囲を推奨しています。
        - ・10.0.0.0/8      （10.0.0.0 ～ 10.255.255.255）
        - ・172.16.0.0/12   （172.16.0.0 ～ 172.31.255.255）
        - ・192.168.0.0/16  （192.168.0.0 ～ 192.168.255.255）
    - Amazon VPCの仕様により、**割り当てられるCIDRブロックサイズは/16～/28のサイズにする必要があります。**
    - IPアドレスが少ない方が立てれるサーバーの数が減ってセキュリティ面で強くなったりしますが、
- `InstanceTenancy: default`:
    - 基本はデフォルトを選んでおこう


# AWS Subnet

VPCを細かく分割したのがサブネットです。
サブネットは「プライベートサブネット」「パブリックサブネット」などインターネットに対して開いている、閉じているなどを設定することができます。

- パブリックサブネット: 直接インターネットアクセスが可能なサブネット。
- プライベートサブネット: インターネットアクセスが制限されているサブネット。
    - また、パブリックサブネットには **インターネットゲートウェイ** と呼ばれるインターネットに接続できるサービスを設定できます。

具体的にはサブネットをプライベート、パブリックたらしめているものは、**ルートテーブルの設定と、その中でのインターネットゲートウェイの使用**
そして、**ネットワークACLの三つ** です。

## SubnetのIaC

```yml
PublicSubnetA:
  Type: AWS::EC2::Subnet
  DeletionPolicy: Delete
  Properties:
    VpcId: !Ref ProductVPC
    AvailabilityZone: "ap-northeast-1a" # ここを指定しないと海外リージョンに作成されてしまうこともある！
    Tags:
    - Key: Name
      Value: PublicSubnetA
```

- `AvailabilityZone: "ap-northeast-1a"`
    - ここの設定は **日本で開発をするのであればなおさら必須** 


## サブネットのCIDRの決め方

サブネットのCIDRの決め方は、VPCのCIDRの範囲内に収まるようにする！ && 利用するIPアドレスの数次第で決まる!

- VPCのCIDRが`192.168.0.0/16`の時
- サブネットのCIDRはその中から切り出すイメージで `192.168.1.0/24`を指定する
- この場合、利用可能なIPアドレスの数が `251` あることを確認する。 ( ネットマスクが24であるため、残りの8バイト => 0~255個使える。 )
- 仮に、この中でEC2インスタンスを立てた場合は `192.168.1.???`に該当するIPアドレスが **プライベートIPアドレス(AWS内部で閉じているIPアドレス)** が割り振られる。
    - EC2インスタンスの**パブリックIPはAWSが適当につけるIPアドレスなので、EC2を起動、停止するたびに変わることに注意**

以下プライベートIPアドレスについてのコラム

> ちなみに、「192.168.x.x」（192.168.0.0～192.168.255.255）の範囲のIPアドレスは，このプライベート・アドレスにあたる。
この範囲のIPアドレスはパブリックなIPアドレスではなく、
その他、プライベートIPアドレスとして使える範囲は決められている。「10.0.0.0～10.255.255.255」「172.16.0.0～172.31.255.255」「192.168.0.0～192.168.255.255」である。どれを使うかは、IPアドレスを割り当てるホスト数で決める。同一ネットワークでなければ重複しても構わない。
コンソール画面から、EC2のプライベートIPアドレスは



# AWSルートテーブルとAWSルート

サブネット内外の通信では、**ルートテーブル**と**ルート**の二つのリソースを使用する。

ルートテーブルはサブネットのIPアドレスに応じて、どのように通信するかorブロックするかを決めるテーブルであり、
ルートはテーブルの行(IPと通信経路の一組の対応)である。

<img src="https://raw.githubusercontent.com/minegishirei/store/main/aws/network/route.webp">

from https://dev.classmethod.jp/articles/subnet-to-subnet-traffic-with-amazon-vpc-more-specific-routing/



## AWSのルートテーブル、ルートのIaCコード

- パブリックサブネットのルートテーブルの例

インターネットゲートウェイが刺さっているのが確認できる。

```yml
RouteTable:
  Type: AWS::EC2::RouteTable
  Properties:
    VpcId: !Ref VPC
    Tags:
      - Key: Name
        Value: PublicRouteTable

PublicRoute:
  Type: AWS::EC2::Route
  Properties:
    RouteTableId: !Ref RouteTable
    DestinationCidrBlock: 0.0.0.0/0
    GatewayId: !Ref InternetGateway
```

- プライベートサブネットのルートテーブルの例

以下の例では、Natゲートウェイを使用してインターネットに接続している例です。

```yml
RouteTable:
  Type: AWS::EC2::RouteTable
  Properties:
    VpcId: !Ref VPC
    Tags:
      - Key: Name
        Value: PrivateRouteTable

PrivateRoute:
  Type: AWS::EC2::Route
  Properties:
    RouteTableId: !Ref RouteTable
    DestinationCidrBlock: 0.0.0.0/0
    NatGatewayId: !Ref NatGateway
```

ちなみに、サブネットとルートテーブルを結びつけるためには `SubnetRouteTableAssociation` リソースを使用します。

```yml
PublicSubnetRouteTableAssociation:
  Type: AWS::EC2::SubnetRouteTableAssociation
  Properties:
    SubnetId: !Ref PublicSubnet
    RouteTableId: !Ref PublicRouteTable
```


# AWS InternetGateway

インターネットゲートウェイはパブリックサブネットとインターネットを繋げるサービスです。
以下のように、本体である `InternetGateway`と `VPCGatewayAttachment`を揃えることでVPCにアタッチすることができます。

```yml
InternetGW:
  Type: AWS::EC2:InternetGateway
  DeletionPolicy: Delete
  Propaties:
    Tags:
    - Key: Name
      Value: PublicIGW

AttachmentIGW:
  Type: AWS::EC2::VPCGatewayAttachment
  DeletionPolicy: Delete
  Properties:
    InternetGatewayId: !Ref InternetGW
    VpcId: !Ref ProductVPC
```

これだけだとVPC内部にインターネットゲートウェイ存在するだけなので、 Pubilcルートテーブルと繋げる必要があります。
以下は Routeリソースを使用して `0.0.0.0/0` に対応するリソースとして `InternetGateway`を設定することでインターネットと接続することが可能です。

```yml
PublicRoute:
  Type: AWS::EC2::Route
  Properties:
    RouteTableId: !Ref PublicRouteTable
    DestinationCidrBlock: 0.0.0.0/0
    GatewayId: !Ref InternetGateway
```


# AWS NetworkACL

ネットワーク内部での動きを制限することが可能です。
NetworkACLはNACL(ナクル)とも呼ばれます。

NACLをサブネットに作成すると以下のコンソール画面のように **サブネットと外部の通信を制限することが可能です。**

次のサブネットは `0.0.0.0/0`(インターネットのIPのこと)から入ってくる通信で、80,443,22を許可してます。
その他、`10.0.0.0/16` は全ての通信を許可していますが、これはプライベートIPアドレスのことを指しています。

<img src="https://i.stack.imgur.com/zzeqI.png">


## パブリックなNetwork ACLの例

- インバウンドルール

| ルール番号 | 許可/拒否 | プロトコル | ポート範囲 | ソース | 説明  |
| --- | --- | --- | --- | --- | --- |
| 100 | 許可  | TCP | 22  | 203.0.113.0/24 | SSHアクセス |
| 110 | 許可  | TCP | 80  | 0.0.0.0/0 | HTTPアクセスの許可 |
| 120 | 許可  | TCP | 443 | 0.0.0.0/0 | HTTPSアクセスの許可 |
| \*  | 拒否  | 全て  | 全て  | 全て  | デフォルトの拒否ルール |

- アウトバウンドルール

| ルール番号 | 許可/拒否 | プロトコル | ポート範囲 | デスティネーション | 説明  |
| --- | --- | --- | --- | --- | --- |
| 100 | 許可  | TCP | 80  | 0.0.0.0/0 | HTTPアクセスの許可 |
| 110 | 許可  | TCP | 443 | 0.0.0.0/0 | HTTPSアクセスの許可 |
| 120 | 許可  | UDP | 53  | 0.0.0.0/0 | DNSクエリの許可 |
| \*  | 拒否  | 全て  | 全て  | 全て  | デフォルトの拒否ルール |


上記のルールは次のような意味を表しています。

- SSH トラフィックの許可: 管理目的でSSH(22)接続を許可しますが、特定のIPアドレス範囲からのみに制限します（例: 管理者のIPアドレスのみ）。今回は(203.0.113.0/24)を許可しています。
  - 今回インバウンドルールでSSHを許可しているため、外部のPCから接続することはできますが、アウトバウンドでは拒否されています。
    これにより、「内部からSSH接続を使用して踏み台サーバーとして利用されること」を防いでいます。
- HTTP/HTTPS トラフィックの許可: Webサーバーが配置されることが多いため、HTTP(80)およびHTTPS(443)のトラフィックを許可します。

その他、デフォルトはすべて拒否している状態です。


## プライベートなNetworkAClの例

- インバウンドルール

| ルール番号 | 許可/拒否 | プロトコル | ポート範囲 | ソース | 説明  |
| --- | --- | --- | --- | --- | --- |
| 100 | 許可  | TCP | 22  | 203.0.113.0/24 | SSHアクセス |
| 110 | 許可  | TCP | 3306 | 10.0.0.0/16 | データベースアクセス（MySQL） |
| 120 | 拒否  | TCP | 80  | 0.0.0.0/0 | HTTPアクセスの拒否 |
| 130 | 拒否  | TCP | 443 | 0.0.0.0/0 | HTTPSアクセスの拒否 |
| \*  | 拒否  | 全て  | 全て  | 全て  | デフォルトの拒否ルール |

- アウトバウンドルール

| ルール番号 | 許可/拒否 | プロトコル | ポート範囲 | デスティネーション | 説明  |
| --- | --- | --- | --- | --- | --- |
| 100 | 許可  | UDP | 53  | 0.0.0.0/0 | DNSクエリ |
| 110 | 拒否  | TCP | 80  | 0.0.0.0/0 | HTTPアクセスの拒否 |
| 120 | 拒否  | TCP | 443 | 0.0.0.0/0 | HTTPSアクセスの拒否 |
| \*  | 拒否  | 全て  | 全て  | 全て  | デフォルトの拒否ルール |





## ネットワークACLのIaCコードの例


```yml
PublicNetworkACL:
  Type: AWS::EC2::NetworkAcl
  DeletionPolicy: Delete
  Properties:
    VpcId: !Ref ProductVPC
    Tags:
    - Key: Name
```







# セキュリティグループ

これまでは「ネットワーク」の話をしてきましたが、セキュリティグループは「リソース」に対するアクセス制御です。

- NetworkACLがサブネットの制限を行うのに対して
- セキュリティグループはリソースの制限を行います


## HTTP/HTTPSサーバー向けのセキュリティグループ

- インバウンドルール:
  - タイプ: HTTP (ポート80)、HTTPS (ポート443)
  - プロトコル: TCP
  - 許可ポート: 80, 443
  - ソース: 0.0.0.0/0 (どこからでもアクセス可能)
- アウトバウンドルール:
  - 通常はデフォルトですべてのトラフィックを許可するように設定されます。


このセキュリティグループは、WebサーバーなどのHTTPやHTTPSを使用するリソースに適しています。

| タイプ | プロトコル | ポート範囲 | ソース |
| --- | --- | --- | --- |
| HTTP | TCP | 80  | 0.0.0.0/0 |
| HTTPS | TCP | 443 | 0.0.0.0/0 |
| アウトバウンド | すべて | \-  | \-  |

パブリックなリソースではデータベースのような「外向きに出たら困るもの」はそこまで置いてない(?)ため、
アウトバウンドは許可される傾向にあります。


## サブネットに合わせるケース

以下はPublicなサブネット内部のリソースに紐付ける想定のサブネットです。

| タイプ | プロトコル | ポート範囲 | ソース |
| --- | --- | --- | --- |
| Public-AZ01 | 全て | 全て  | 172.16.0.0.0/20 |
| Public-AZ02 | 全て | 全て | 172.16.16.0/20 |
| Public-AZ03 | 全て | 全て | 172.16.32.0/20 |

サブネット内部の通信を許可するために、上記のセキュリティグループを紐づけても良いでしょう。










