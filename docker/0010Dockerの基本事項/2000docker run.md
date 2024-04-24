

[:contents]



## docker run とは何か?

Dockerコンテナの実行で最も基本的なコマンドが`docker run`コマンドです。
**新しくコンテナを立ち上げ、コンテナ内で指定したコマンドを実行することができます。**

`docker run`コマンドの構文は以下の通りで、<イメージ名>で指定したイメージからコンテナを立ち上げて、その中で<コマンド>を実行します。

```sh
docker run  <オプション(-または--で始まる)>  <イメージ名>  <コマンド>
```

例えば、`debian`イメージ（linuxの最も基本的なOSディストリビューションの一つ）で`echo "hello world"`と実行したければ以下のように`run`コマンドを実行します。

```sh
docker run debian echo "Hello world"
```

このように、 **`docker run`コマンドは指定したイメージからコンテナを立ち上げ、任意のコマンドを実行することができます。** 



## docker run のイメージ名の指定について

`docker run`コマンドで使用する<イメージ名>は、起動したいDockerイメージを指定します。

```sh
docker run  <オプション(-または--で始まる)>  <イメージ名>  <コマンド>
```

この<イメージ名>にはいくつかの指定方法があります。

- 公式イメージ
    - Docker Hubなどのイメージレジストリから提供されている公式イメージは、単純にイメージ名を指定するだけで利用できます。
    - 例えば、debianやnginxなどが公式イメージの一例です。

```sh
docker run debian
docker run nginx
```

- カスタムイメージ
    - ユーザーが自身で作成したカスタムイメージは、そのイメージの名前を指定します。通常、ユーザー名や組織名と共にイメージ名が指定されます。

```sh
docker run myusername/myimage
```

- タグ付きイメージ
    - 同じイメージ名でも、バージョンやタグによって異なるバージョンのイメージを指定することができます。
    - デフォルトでは、latestというタグが使用されます。

```sh
docker run myimage:latest
docker run myimage:v1.0
```


- Dockerレジストリの指定
    - イメージがDocker Hub以外のレジストリにある場合は、そのレジストリのURLを含めて指定します。
    - 次の例は`AWS Public Gallery`というAWSが提供するパブリックレジストリのurlで、mysqlを指定しています。

```sh
docker run public.ecr.aws/docker/library/mysql:oraclelinux8
```

これらの方法を使用して、docker runコマンドで適切なイメージを指定し、コンテナを起動することができます。






## docker runでbashに接続する

`docker run`コマンドは任意のコマンドを実行できますが、`bash`や`python3` などの対話型のサービスを起動することもできます。
ただし、対話型のサービスを立ち上げるためには`-it`オプションを指定する必要があります。

```sh
docker run -it  debian bash
```

上記のコードを実行すると、debianイメージのコンテナが起動しその中でbashが起動します。
これによりコンテナ内でシェルを対話的に操作することができるため、コンテナ内での作業やデバッグ、設定変更などが可能になります。

また、debianイメージ以外のイメージを使用しても同様に対話的な接続を行うことができます。
例えば、Pythonの対話型インタプリタを起動するには、以下のようにpython3コマンドを指定します。

```sh
docker run -it python:3
```

これにより、Pythonの対話型インタプリタが起動し、コンテナ内でPythonのコードを対話的に実行できるようになります。



## docker runとcreate の違い

`docker create`コマンドも `docker run`コマンドも「コンテナを作成する」という点については同じ挙動をします。
しかし、`docker create`コマンドはコンテナを作成するのみで、コマンドの実行は行いません。
その代わり、`docker create`コマンドの実行後はコンテナIDを生成し画面に表示します。

```sh
PS C:\Users\mineg> docker create -it ubuntu bash
155e2b8a63cbb56dccbf6b707b018cb24bd7cb18cec055d35c1746d6f87df917
```

一度生成されたコンテナは`docker start`コマンドで実行することが可能です。

```sh
PS C:\Users\mineg> docker start -i 155e2b8a63cbb56dccbf6b707b018cb24bd7cb18cec055d35c1746d6f87df917
root@155e2b8a63cb:/# ls
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@155e2b8a63cb:/#
```


## よく使うオプション

### 1. インタラクティブモードとバックグラウンドモードの違い

docker run コマンドを使用する際には、インタラクティブモードとバックグラウンドモードのどちらを選択するかを考慮する必要があります。

- インタラクティブモード: -i オプションを指定することで、コンテナの標準入力にアクセスし、直接操作することができます。このモードでは、コンテナ内のシェルに対話的にアクセスできます。

```sh
docker run -i ubuntu bash
```

- バックグラウンドモード: -d オプションを使用すると、コンテナをデタッチし、バックグラウンドで実行します。これにより、コンテナの実行中にコマンドプロンプトを解放し、ユーザーが他の作業を続けられるようになります。

```sh
docker run -d ubuntu bash
```


### 2. ボリュームのマウント

docker run コマンドを使用してコンテナを起動する際に、ボリュームをホストマシンのディレクトリにマウントすることができます。これにより、コンテナとホスト間でデータを共有し、永続的なストレージを実現できます。

```sh
docker run -v /host/directory:/container/directory ubuntu bash
```

### 3. 環境変数の設定

docker run コマンドを使用してコンテナを起動する際に、環境変数を指定することができます。-e オプションを使用することで、コンテナ内で使用する環境変数を設定し、アプリケーションの動作をカスタマイズすることができます。

```sh
docker run -e VARIABLE_NAME=variable_value ubuntu bash
```

### 4. CPUやメモリの制限

デフォルトでは、Dockerコンテナはホストマシンのリソースを共有しますが、docker run コマンドを使用してコンテナに割り当てるCPUやメモリの量を制限することができます。これにより、コンテナがホストマシンのリソースを過剰に消費することを防ぎ、アプリケーションのパフォーマンスを制御できます。

```sh
docker run --cpus=2 --memory=4g ubuntu bash
```

以上が、docker run コマンドの主な実行オプションに関する概要です。これらのオプションを適切に使用することで、Dockerコンテナの起動と実行をより効果的に管理できます。







## NVIDIA GPU にアクセスする

この--gpusフラグを使用すると、NVIDIA GPU リソースにアクセスできるようになります。まず、 nvidia-container-runtimeをインストールする必要があります。 詳細については、「コンテナーのリソースの指定」を参照してください。

を使用するには--gpus、使用する GPU (またはすべて) を指定します。値を指定しない場合は、使用可能なすべての GPU が使用されます。以下の例では、利用可能なすべての GPU を公開しています。

 docker run -it --rm --gpus all ubuntu nvidia-smi
このdeviceオプションを使用して GPU を指定します。以下の例では、特定の GPU を公開しています。

 docker run -it --rm --gpus device=GPU-3a23c669-1f69-c64e-cf85-44e9b07e7a2a ubuntu nvidia-smi
以下の例では、1 番目と 3 番目の GPU を公開しています。

 docker run -it --rm --gpus '"device=0,2"' nvidia-smi


https://matsuand.github.io/docs.docker.jp.onthefly/engine/reference/commandline/run/











page:https://minegishirei.hatenablog.com/entry/2024/04/09/180426


