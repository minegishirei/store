New Relic APM徹底解説とPython実装ガイド
NewRelic
6802418398344200164


tag:auto

[:contents]





## New Relic APM徹底解説とPython実装ガイド

### はじめに

モダンなアプリケーションを「インフラ」「バックエンド」「フロントエンド」と分割した時に、NewRelicAPMは、バックエンドをオブザーバビリティを達成するツールになります。

### 主な機能

- トランザクションごとの応答時間、スループット、エラー率の可視化による問題のある処理の特定
- DB、外部API との通信状況の分析
- エラー・例外の自動検出とグループ化
- **リリース前後のパフォーマンス比較**
- **ブレークダウンによる処理群のスライス。**
- **分散トレーシングによるマイクロサービス間を横断した遅延分析** (サービスマップと分散トレーシング)
- リアルタイムアラートとSlack等との連携


### ブレークダウン

APMの場合、特にすごいのはブレークダウン機能です。

New Relic APMのブレークダウン機能は、アプリケーション内で発生する各トランザクションやリクエストの処理時間を詳細に分解・可視化します。
これにより、どの処理（例：データベースアクセス、外部API呼び出し、テンプレートレンダリングなど）がボトルネックになっているかを一目で把握できます。



- 主なポイント
    - 各トランザクションの処理ステップごとの所要時間をグラフで表示
    - 外部サービスとの通信やDBクエリの遅延を個別に分析
    - コードレベルでどの関数・メソッドが時間を消費しているかを特定
    - パフォーマンス改善のための優先的な対応箇所を明確化

この機能を活用することで、アプリケーションのパフォーマンス課題を迅速に発見し、効率的なチューニングや障害対応が可能となります。

<img src="https://github.com/minegishirei/store/blob/main/sre/breakedown/image.png?raw=true" />


### サービスマップ

分散トレーシングは、**1つのリクエスト（トレース）が複数のサービスを経由して処理される流れを追跡する手法**です。
リクエストはしばしばプロセスやマシン、ネットワークの境界を越えて伝播します。
特にマイクロサービスアーキテクチャの普及により、どこで障害や遅延が発生しているのかを特定するために、分散トレーシングの重要性が高まっています。
オンプレミスからクラウド、管理外のSaaSサービスなど、さまざまな境界をまたぐリクエストの問題を特定し、パフォーマンスの最適化や信頼性向上に役立ちます。

> **Note:分散トレーシングの実装例**
> 
> 分散トレーシングの普及に伴い、さまざまなアプローチや標準が登場しました。
> 2010年にGoogleがDapper論文を発表したことで注目され、TwitterのZipkin（2012年）、UberのJaeger（2017年）などのオープンソースプロジェクトや、Honeycomb、Lightstepといった商用サービスも登場しています。
> 実装方法は異なりますが、分散トレーシングの基本的な価値や目的は共通しています。

現代の分散システムは多くの依存関係が絡み合い、全体像を把握するのが困難です。
分散トレーシングは、各サービスやコンポーネント間の関係性を可視化し、システム全体の依存関係を明確にします。
例えば、下流のデータベースで発生した遅延が、上流の複数サービスに波及する場合、どこが根本原因なのかを特定するのは容易ではありません。
分散トレーシングを活用することで、こうした複雑な問題の解明が可能となります。

オブザーバブルなシステムにおいて、トレースは相互に関連付けられた一連のイベントとして記録されます。
それらのイベント情報をもとに作られるマップが「サービスマップ」です。

<img src="https://github.com/minegishirei/store/blob/main/sre/trace/image.png?raw=true" />

https://newrelic.com/jp/learn/nru200#backend

上記の画像では「Webブラウザ」からバックエンドのサービスを経由して、他のサービスを呼びだしている様子が確認できます。
**バックエンドだけでなく、フロントエンドなどにもNewRelic Agentを導入することで同一のトレースIDをもとにサービスマップを作成してくれるのです。**


### 分散トレーシング

NewRelicには分散トレーシングと呼ばれる機能があります。
先ほどの、サービスマップはサービス全体を俯瞰して見れる機能ですが、
ユーザーから「特定の画面でエラーが起きた」と報告を受けた際にはこの分散トレーシング機能が役に立ちます。

分散トレーシング機能のトップ画面は以下の通りです。
これを確認すると、イベントの始まりごと(この場合だと、WebアプリケーションのURL)にトレースが分類分けされています。

<img src="https://github.com/minegishirei/store/blob/main/sre/trace/distributed1/image.png?raw=true" />

グループをクリックすると、同じイベントスタートとして分類されたトレースが全て一覧で表示されます。
この場合だと、`browser/iphone.jsp` のURLから始まったイベント全てが確認できます。

問題のあるトレース(たとえば、やたら遅いトレース)をクリックすると、そのトレースにまつわるサービスマップが確認できます。

<img src="https://github.com/minegishirei/store/blob/main/sre/trace/distributed2/image.png?raw=true" />

この画面では、一つのトレースについてのコールツリー/スパンツリー が確認できます。
どういうことかというと、関数の呼び出し関係を、サーバー越しに追いかけることが可能になってます。

<img src="https://github.com/minegishirei/store/blob/main/sre/trace/distributed3/image.png?raw=true" />

トレースツリーの時間がかかっている部分を確認すると、MySQLの特定のクエリで大幅に時間を消費していることが確認できます。

ここまでで重要なことは、 **ブラウザのイベントから始まって、データベースのクエリまで根本原因を特定できたことです。** 
分散トレースの利用により、ユーザー視点から根本原因までたどり着くことが可能なのです。









page:https://minegishirei.hatenablog.com/entry/2025/04/13/173708




流れ

14945776032085753483






## NewRelicにおけるトラブルシューティングの例

Newrelic univercity より抜粋。

1. 障害検知
2. 発生時間帯に絞り、実行時間が長い問題のAPIを特定。
3. トレースデータを見て、原因特定
4. 補助的にログを確認

<img src="https://github.com/minegishirei/store/blob/main/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202026-10-03%2020.30.17.png?raw=true" />

https://newrelic.com/jp/learn

### Change Tracking

NewRelicには Change Tracking という機能があります。
アプリケーションの修正やシステム構成の変更を記録し、アプリケーションのパフォーマンスやエラーの情報を「バックエンド・フロントエンド・インフラ」の横断で見やすくすることができます。

- 「今回の障害発生前にどんなリリースがあったのか?」
- 「今回のデプロイメントがこの問題を引き起こしたのか？」

デプロイメントとパフォーマンス低下との相関関係が分析しやすくなります。

Change Trackingは通常のアプリケーションの登録と同じように実行します。

<img src="https://github.com/minegishirei/store/blob/main/sre/flow/image.png?raw=true" />

Change Trackingがうまく登録できると、デプロイ前後のパフォーマンスがうまく可視化されます。

<img src="https://github.com/minegishirei/store/blob/main/sre/flow/4.png?raw=true" />

**サービスの障害が発生したときやリリース時に、Change Trackingの機能を使用することで、グラフの変化とアクションの関係を見やすくできます。**


### 動的閾値

問題の発生を検知するフローはいくつかありますが、問題の発生確認は動的閾値でも確認が可能。
動的閾値はいつもと異なる振る舞いを検知し、問題の発生(あるいは、問題が発生するよりも前に)アラートで気づくことができます。

<img src="https://github.com/minegishirei/store/blob/main/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202026-10-03%2020.38.48.png?raw=true" />

https://newrelic.com/jp/learn


### 外形監視

問題の発生の検知は動的検知だけではありません。
外形監視を設定していた場合は、より直接的に問題の発生を検知できます。

<img src="https://github.com/minegishirei/store/blob/main/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202026-10-03%2020.24.12.png?raw=true" />

https://newrelic.com/jp/learn












page:https://minegishirei.hatenablog.com/entry/2026/10/04/014951



パフォーマンスの解析を行う

14945776032089713928



## Change Tracking

NewRelicには Change Tracking という機能があります。
アプリケーションの修正やシステム構成の変更を記録し、アプリケーションのパフォーマンスやエラーの情報を「バックエンド・フロントエンド・インフラ」の横断で見やすくすることができます。

- 「今回の障害発生前にどんなリリースがあったのか?」
- 「今回のデプロイメントがこの問題を引き起こしたのか？」

デプロイメントとパフォーマンス低下との相関関係が分析しやすくなります。

Change Trackingは通常のアプリケーションの登録と同じように実行します。

<img src="https://github.com/minegishirei/store/blob/main/sre/flow/image.png?raw=true" />

Change Trackingがうまく登録できると、デプロイ前後のパフォーマンスがうまく可視化されます。

<img src="https://github.com/minegishirei/store/blob/main/sre/flow/4.png?raw=true" />

**サービスの障害が発生したときやリリース時に、Change Trackingの機能を使用することで、グラフの変化とアクションの関係を見やすくできます。**



## APM Web transactions time

NewRelic APMの `Web transactions time` がパフォーマンス解析の第一歩です。

この画面を開くことで、グラフの時系列とどこでアクセス数が伸びたのかのパターンを確認することが可能です。

<img src="https://github.com/minegishirei/store/blob/main//sre/perfomance/image.png?raw=true" />

https://docs.newrelic.com/jp/docs/tutorial-improve-app-performance/root-causes/


上記の例だと特段大きな問題は抱えておらず、グラフの変化も大きなものはないのですが **RubyとWebExternalに大部分の時間が使われていることがわかります**。

- Rubyはその通り、Rubyでの処理時間のこと
- Web Externalは外部APIの呼び出しのこと

また、データベースのパフォーマンスに問題がある時は「MySQL」「PostgreSQL」などのデータベース名が出てきます。


## パフォーマンスで目指すべきところは APM Transaction

パフォーマンスで目指すべきところは APM Transactionで特定のトランザクションを探し出すことです。

`APM Transaction` を用いることで根本原因のドリルダウンができます。

<img src="https://github.com/minegishirei/store/blob/main/sre/perfomance/image2.png?raw=true" />

特に、トランザクショントレースのウォーターフォールビューは、一発で遅延箇所がわかります。

そして、`Slow Span` にチェックを入れることで、より直接的にパフォーマンス劣化の原因がわかります。

<img src="https://github.com/minegishirei/store/blob/main/sre/perfomance/handson4/image.png?raw=true" />



## 遅いトランザクションを特定する

Transaction > フィルタリングで `Slowest average time` 

<img src="https://github.com/minegishirei/store/blob/main/sre/perfomance/handson1/image.png?raw=true" />

これにより遅いトランザクションを特定する。
これで最も遅い処理が最上部に来るので、クリックし

<img src="https://github.com/minegishirei/store/blob/main/sre/perfomance/handson2/image.png?raw=true" />

詳細を確認できる。

下にスクロールすると、同一カテゴリのトランザクションの中でもさらに遅いものが表示されるので、それをクリックする。

<img src="https://github.com/minegishirei/store/blob/main/sre/perfomance/handson3/image.png?raw=true" />

ここまでくると、目指していた APM Transactionでの根本原因のドリルダウンが可能になります。










page:https://minegishirei.hatenablog.com/entry/2026/10/13/183126
