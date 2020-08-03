# Building Secure Microservices-based Applications Using Service-Mesh Architecture和訳

個人的な勉強のため，NIST SP 800-204a "Building Secure Microservices-based Applications Using Service-Mesh Architecture" を和訳していきます。原文は[こちら](https://csrc.nist.gov/publications/detail/sp/800-204a/final)です。

権限的に問題がありましたら公開を取りやめますので，お手数ですがご連絡ください。

技術的な内容はなるべく文意に沿って翻訳いたしますが，内容の正確さを保証するものではございません。  
また，謝辞など技術に関係ない節は省略しております。

なお，翻訳にあたり，[DEEPL](https://www.deepl.com/home)や[Google Translate](https://translate.google.co.jp/)を活用させていただいております。

---

NIST Special Publication 800-204A

## サービスメッシュアーキテクチャを用いたセキュアなマイクロサービス型アプリケーションの構築

- Ramaswamy Chandramouli, Zack Butcher著
- https://doi.org/10.6028/NIST.SP.800-204A
- 2020年5月

## コンピュータシステム技術に関する報告書

国立標準技術研究所(NIST; National Institute of Standards and Technology)の情報技術研究所(ITL; The Information Technology Laboratory)は，国家の測定・標準基盤において技術的リーダーシップを提供することにより米国経済と公共の福祉を促進するものである。ITLは，情報技術の開発と生産的な利用促進のため，テスト，テスト方法，参照データ，PoC実装，技術的分析を開発するものである。ITLの責任には，連邦政府の情報システムにおいて国家安全保障関連情報以外の情報に対して，費用対効果の高いセキュリティとプライバシーを提供するための管理，運営，技術，物理における標準規格およびガイドラインの開発を含んでいる。Special Publication 800シリーズは，情報システムセキュリティにおけるITLの研究やガイドライン，支援活動，および産業界，政府，学術機関との協力活動を報告するものである。

## 要旨

構築の傾向が高まっているマイクロサービス型アプリケーションは，そのユニークな特徴から，サービス間の相互作用についてあらゆる側面からセキュリティ的な対処が必要である。
分散した複数ドメインをマイクロサービスの性質から，セキュアなトークンサービス（STS; Secure Token Service），認証・認可のための鍵管理および暗号化のサービス，およびセキュアな通信プロトコルが必要である。
（マイクロサービスを構成する）クラスタ化されたコンテナは短命であるため，セキュアなサービスディスカバリが必要である。
同様に，可用性には (a)ロードバランシング，サーキットブレイキング，スロットリングのような回復力の技術，(b)（サービス安定稼働のための）継続的な監視，という要件がある。*サービスメッシュ*は，これら要件を抽象的なレベルで容易に指定できるようにする最もよく知られたアプローチであり，均一かつ一貫性を持った定義ができる上，個々のマイクロサービスのコードの変更なしで組み込めるものである。本文書の目的は，マイクロサービス型アプリケーション支援のため，頑強なセキュリティ基盤を形成するプロキシベースのサービスメッシュコンポーネントの構築ガイダンスを提供することである。

## キーワード

APIゲートウェイ，アプリケーション プログラミング インタフェース(API)，サーキットブレイカー，ロードバランシング，マイクロサービス，サービスメッシュ，サービスプロキシ

## 注意 特許開示通知

情報技術研究所（ITL）は，この出版物のガイダンスまたは要件に準拠するために使用が必要となる可能性のある特許クレームの所有者に，特許クレームをITLへ開示することを要求している。ただし，特許の保有者は，ITLの特許請求に応答する義務はなく，ITLはこの出版物に適用される可能性のある特許を特定する調査を行っていない。出版日現在，また本書のガイダンスまたは要件に準拠するために使用が必要となる可能性のある特許クレームを特定するための募集を行った時点は，このような特許クレームはITLに確認されていない。 ITLは，この出版物の使用における特許侵害を回避するためにライセンスが必要でないことを表明または暗示するものではない。

## エグゼクティブサマリ

マイクロサービスベースのアプリケーション・アーキテクチャは，そのスケーラビリティ，デプロイの俊敏性，ツールの可用性の高さから，クラウドベースのアプリケーションや大規模なエンタープライズ・アプリケーションを構築する際の規範となりつつある。同時に，マイクロサービスベースのアプリケーションの特性は，セキュリティ要件の変更や強化をもたらす。

これら特性とそのセキュリティへの影響におけるいくつか例は次のとおりである。

1. マイクロサービスの数が膨大になると，相互接続が多くなり，保護すべき通信リンクも増加する
2. マイクロサービスの短命の性質は，安全なサービスディスカバリの仕組みを必要とする
3. 細分化されたマイクロサービスの性質は，きめ細かな認可ポリシーを必要とする

マイクロサービスベースのアプリケーションのためにサポートされるサービス（認証/認可，セキュリティ監視など）は，サービスメッシュのような専用のインフラを介してしっかりと調整されなければならない。サービスメッシュのコンポーネントをデプロイする方法は複数ある。アプリケーション（マイクロサービス）のコードに組み込む方法，ライブラリとして実装してアプリケーションのコードに結合する方法，アプリケーションコードから独立したサービスプロキシとして実装する方法などである。最後のデプロイ方法は，マイクロサービスベースのアプリケーションを支援するインフラを実装する多くのシナリオにおいて，スケーラビリティと柔軟性の点で最も効率的であることがわかっている。

本ドキュメントの目的は，サービスプロキシベースのアプローチにおけるサービスメッシュ・コンポーネントのデプロイガイダンスを提供することである。サービスメッシュのデプロイにおける推奨事項は以下の通りである

- サービスプロキシ間の通信設定
- イングレスプロキシの設定
- 外部サービスへのアクセス設定
- IDやアクセス管理の設定
- 監視機能の設定
- ネットワークレジリエンスの設定
- オリジン間リソース共有（CORS）の設定
- 管理者権限の設定

## 1 はじめに

マイクロサービスアーキテクチャはエンタープライズやクラウドベースのアプリケーションを構築するための確立されたアプローチになっている。これは次の理由からである:

- 俊敏性: マイクロサービス群の疎結合性とモジュール性の向上により，他のコンポーネント（マイクロサービス）に影響を与えることなく独立的に迅速な修正やデプロイが可能となる
- スケーラビリティ: マイクロサービスの特性により，個々のマイクロサービスを独立に拡張できる
- 利便性: 明確に定義されたAPIを使用し，様々はマイクロサービスの統合や受け入れが可能となる
- ツールの可用性: 自動化ツールの可用性の向上はエラーのない設定とデプロイを容易になる

上記の利点にもかかわらず，マイクロサービスベースのアプリケーションアーキテクチャにはセキュリティ要件の修正や強化を伴う以下の課題がある。

- マイクロサービスが多くなるに伴い，コンポーネント間の接続や保護すべき通信リンクが増加する
- コンポーネント（マイクロサービス）が動的に生成・削除されるため，セキュアなサービスディスカバリが必要となる
- ネットワークの境界線という概念がない
- すべてのマイクロサービスは信頼できないものとして扱わなければならない
- マイクロサービスのきめ細かな性質は，マイクロサービスごとの細かな認証を必要とする。それには，セキュリティポリシーを一元的に定義し，その設定を各マイクロサービスに反映することで，マイクロサービス全体に渡って統一的かつ一貫性のある運用を可能にする必要がある

### 1.1 何故サービスメッシュか (Why Service Mesh)

上述のマイクロサービスベースのアプリケーションセキュリティ要件のため，アプリケーションを補助するインフラと，そのインフラに関連するサービス（例えばセキュリティ）は，緊密に連携されている必要がある。
サービスメッシュはそのような専用のインフラの一つである。
サービスメッシュを実現するコードは，マイクロサービスベースのアプリケーションアーキテクチャのコンポーネントに関して，以下のように体系づけられる（各アーキテクチャパターンは，SM-ARxという頭字語を用いて表記されます（SMはサービスメッシュ，ARはアーキテクチャ，xはシーケンス番号）。

- SM-AR1: サービスメッシュのコードはマイクロサービスアプリケーションのコードに組み込むことができ，アプリケーション開発フレームワークにとって不可欠な部分となる。
- SM-AR2: サービスメッシュのコードはライブラリとして実装され，アプリケーションはAPI呼び出しを介してサービスメッシュが提供する機能に結合される
- SM-AR3: サービスメッシュの機能はプロキシ内に実装されている。各プロキシはマイクロサービスインスタンス(訳者注:コンテナなど個々のマイクロサービスを構成する実態)の前に配置され，マイクロサービスベースのアプリケーションにまとまったインフラサービスを提供する。このプロキシは「サイドカープロキシ」と呼ばれ，アプリケーションコードから独立した実装・運用が可能である。サイドカープロキシは種々のプラットフォーム（様々なプログラミング言語やアプリケーション開発フレームワーク）にとっての最小公分母なAPIであるネットワークを用いることにより，これらの一貫した制御を可能としている。
- SM-AR4: サービスメッシュの機能はプロキシに実装されるが，SM-AR3のようなマイクロサービスインスタンスごとではなく，ノード（物理ホスト）ごと配置される

### 1.2 スコープ

本文書の目的のため，サービスメッシュのアーキテクチャはSM-AR3の構成のみを検討する。
この専用インフラレイヤはアプリケーションのサービスコードを修正することなく，マイクロサービスベースのアプリケーションにあらゆるセキュリティ機能を提供する。
SM-AR4と比べ，SM-AR3は特権のエスカレーション範囲やノイジーネイバー問題（訳者注:高負荷なインスタンスが周辺のインスタンスの性能に影響を及ぼすこと）が少ない。これはSM-AR3がマイクロサービスインスタンスごとに一つのサービスプロキシのインスタンスを配置しており，アプリケーションのトラフィックが専用のサービスプロキシによってのみ中継されることを保証するためである。
SM-AR1およびSM-AR2と比較して，SM-AR3はサービスメッシュの機能を提供するモジュールをアプリケーションのライフサイクルから分離できるため，SM-AR2で発生するような複数言語を跨いで多数のバージョンのライブラリを保守しなければならない組合せ爆発の問題を回避できる。
この文脈に基づき，本文書の観点から見たサービスメッシュの主機能は，エージェントや機能モジュールがマイクロサービスのコードと密結合にならないようにしつつ，クライアントからマイクロサービス，およびマイクロサービスからマイクロサービスへの通信の取次（mediate）および仲介（broker）することである。

### 1.3 対象読者

サービスメッシュフレームワークを用いてマイクロサービスベースのアプリケーションを支援する本ガイダンスの対象読者は，マイクロサービスベースのアプリケーションのためのセキュリティフレームワークを設計したいセキュリティソリューション設計者や，企業やクラウドに存在するさまざまなマイクロサービスベースのアプリケーションのための共通のインフラストラクチャサービスフレームワークを構築するシステムインテグレータが含まれる。

### 1.4 他NISTガイダンス文書との関係

本ガイダンス文書は、マイクロサービスベースのアプリケーションのための特定セキュリティフレームワークやインフラストラクチャの構築に焦点を当てている。マイクロサービスベースのアプリケーションの特性と、その全体的なセキュリティ要件や戦略の理解に有益であり，情報はNIST特別刊行物（SP）800-204「[マイクロサービスベースのアプリケーションシステムのセキュリティ戦略\[1\]][1]」にて提供される。

### 1.5 本文書の構成

本文書の構成は以下の通りである。

- 2章では，[\[1\]][1]で述べたマイクロサービスベースのアプリケーションのセキュリティ要件を参照し，マイクロサービスベースのアプリケーションのセキュリティ要件を再確認する。
- 3章では，サービスメッシュを紹介し，コンポーネント，機能，マイクロサービスベースのアプリケーションにおける通信ミドルウェアとしての独自の役割の概説を述べる。
- 4章では，サービスプロキシ，イングレスプロキシ，イグレスプロキシ，IDとアクセス管理，監視機能，ネットワーク回復力の技術，オリジン間リソース共有(CORS)等の構成領域にまたがるサービスメッシュコンポーネントの詳細なデプロイ推奨事項を述べる。
- 5章では，まとめと結論を述べる。

## 2 マイクロサービスベースのアプリケーションの背景およびセキュリティ要件

マイクロサービスベースのアプリケーションの定義と詳細，脅威およびその対策となるセキュリティ戦略はNIST SP 800-204，[マイクロサービスベースのアプリケーションシステムのセキュリティ戦略\[1\]][1]にて述べられている。
本章の目的はこの種のアプリケーションのセキュリティ要件を再確認および詳述し，第3章で述べるサービスメッシュの機能により，これら要件がどのように満たされるか背景を提供することである。
これにより，第4章の要件を満たすサービスメッシュコンポーネントのデプロイに関する推奨事項の開発が容易になる。

### 2.1 認証・認可の要件

認証およびアクセスポリシーは，マイクロサービスが公開するAPIの種類によって異なる場合がある。これにはパブリックなAPI，プライベートなAPI，パートナーAPI（ビジネスパートナーのみ利用可能なAPI）などがある。マイクロサービスは複数存在し，認証ポリシーはそれらすべてをカバーするように定義される必要がある。更に，証明書ベースの認証は，証明書の生成・管理および鍵管理のために公開鍵インフラ（PKI; Public Key Ingrastracture）を必要とする。その上，全てのマイクロサービスのリソースをカバーする認可モジュールを構築して全てのサービスリクエストにきめ細かな認可を提供しなければならない。

### 2.2 サービスティスカバリ

レガシーな分散システムの場合，複数のサービスは指定されたロケーション（IPアドレスおよびポート番号）で稼働するよう設定されていた。マイクロサービスベースのアプリケーションには以下のようなシナリオが存在するため，堅牢なサービスディスカバリの仕組みが求められる。

1. 相当数のサービスが存在する。各サービスは多数のインスタンスを持ち，，それらは動的にロケーションが変化する
2. 各マイクロサービスは仮想マシン(VM; Virtual Machine)やコンテナとして実装されている場合があり，IPアドレスが動的に割り当てられているかもしれない。特にIaaS(Infrastructure as a Service)やSaaS(Software as a Service)のようなクラウドサービスでは顕著である。
3. オートスケーリングなどの機能により，サービスに紐付けられたインスタンスの数が負荷に応じて変化する

以上の特徴から，サービスリクエストを行いながらサービスを検出する機能は必須の要件となる。本機能を実現する一般的なアプローチとしてサービスレジストリの利用がある。サービスレジストリとは，マイクロサービスベースのアプリケーションのために作成されたサービスインスタンスを登録し，オフラインとなったサービスインスタンスを削除するディレクトリで構成される。

### 2.3 ネットワーク回復性の技術を用いた可用性の改善



## 参考

1. Chandramouli R (2019) Security Strategies for Microservices-based Application Systems. (National Institute of Standards and Technology, Gaithersburg, MD), NIST Special Publication (SP) 800-204. https://doi.org/10.6028/NIST.SP.800-204
2. Indrasiri K (2017) Service Mesh for Microservices. Available at https://medium.com/microservices-in-practice/service-mesh-for-microservices-2953109a3c9a
3. Lerner A, Thomas A (2018) Innovation Insight for Service Mesh. Available athttps://www.gartner.com/en/documents/3894156/innovation-insight-for-service-mesh
4. Morgan W (2017) What’s a service mesh? And why do I need one? Available athttps://buoyant.io/2017/04/25/whats-a-service-mesh-and-why-do-i-need-one/#_ga=2.39876518.581209135.1575405453-1411561046.1575405453
5. Mishra A (2017) Smart Networking with Consul and Service Meshes. Available at.https://www.hashicorp.com/blog/smart-networking-with-consul-and-service-meshes/
6. Schiesser M (2019) Comparing Service Meshes: Linkerd vs. Istio. Available at https://medium.com/glasnostic/comparing-service-meshes-linkerd-vs-istio-c7e0132578a8
7. Bryant D (2018) The Importance of Control Planes with Service Meshes and Front Proxies. Available at https://blog.getambassador.io/the-importance-of-control-planes-with-service-meshes-and-front-proxies-665f90c80b3d
8. Kirschner E (2017) Proxy Based Service Mesh. Available at https://medium.com/@entzik/proxy-based-service-mesh-96cd4b74c198
9. Tiwari A (2017) A sidecar for your service mesh. Available at https://www.abhishek-tiwari.com/a-sidecar-for-your-service-mesh/
10. Sridharan C (2018) The Three Pillars of Observability. Distributed Systems Observability(O’Reilly Media, Inc., Sebastopol, CA), Chapter 4. Available at    https://www.oreilly.com/library/view/distributed-systems-observability/9781492033431/ch04.html
11. Jaeger Project (2020) Jaeger: open source, end-to-end distributed tracing. Available at https://www.jaegertracing.io/
12. Zipkin Project (2020) Zipkin. Available at https://zipkin.io/
13. Leong A (2019) A guide to distributed tracing with Linkerd. Available at https://linkerd.io/2019/10/07/a-guide-to-distributed-tracing-with-linkerd/
14. Stafford GA (2019) Kubernetes-based Microservice Observability with Istio Service Mesh: Part 1 (ITNEXT). Available at https://itnext.io/kubernetes-based-microservice-observability-with-istio-service-mesh-part-1-  bed3dd0fac0b
15. Stafford GA (2019) Istio Observability with Go, gRPC, and Protocol Buffers-based Microservices (ITNEXT). Available at. https://itnext.io/istio-observability-with-go-grpc-and-protocol-buffers-based-microservices-d09e34c1255a

[1]:https://doi.org/10.6028/NIST.SP.800-204
[2]:https://medium.com/microservices-in-practice/service-mesh-for-microservices-2953109a3c9a
[3]:https://www.gartner.com/en/documents/3894156/innovation-insight-for-service-mesh
[4]:https://buoyant.io/2017/04/25/whats-a-service-mesh-and-why-do-i-need-one/#_ga=2.39876518.581209135.1575405453-1411561046.1575405453
[5]:https://www.hashicorp.com/blog/smart-networking-with-consul-and-service-meshes/
[6]:https://medium.com/glasnostic/comparing-service-meshes-linkerd-vs-istio-c7e0132578a8
[7]:https://blog.getambassador.io/the-importance-of-control-planes-with-service-meshes-and-front-proxies-665f90c80b3d
[8]:https://medium.com/@entzik/proxy-based-service-mesh-96cd4b74c198
[9]:https://www.abhishek-tiwari.com/a-sidecar-for-your-service-mesh/
[10]:https://www.oreilly.com/library/view/distributed-systems-observability/9781492033431/ch04.html
[11]:https://www.jaegertracing.io/
[12]:https://zipkin.io/
[13]:https://linkerd.io/2019/10/07/a-guide-to-distributed-tracing-with-linkerd/
[14]:https://itnext.io/kubernetes-based-microservice-observability-with-istio-service-mesh-part-1-bed3dd0fac0b
[15]:https://itnext.io/istio-observability-with-go-grpc-and-protocol-buffers-based-microservices-d09e34c1255a