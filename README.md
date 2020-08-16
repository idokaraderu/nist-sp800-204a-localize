## Building Secure Microservices-based Applications Using Service-Mesh Architectureの和訳プロジェクト<!-- omit in toc -->

勉強のため，NIST SP 800-204a "Building Secure Microservices-based Applications Using Service-Mesh Architecture" を和訳します。原文は[こちら](https://csrc.nist.gov/publications/detail/sp/800-204a/final)です。

権限的に問題がある場合は公開を取りやめますので，お手数ですがご指摘下さい。

技術的な内容はなるべく文意に沿って翻訳いたしますが，内容の正確さを保証するものではございません。また，謝辞など技術に関係ない節は省略しております。

なお，翻訳にあたり，[DEEPL](https://www.deepl.com/home)や[Google Translate](https://translate.google.co.jp/)を活用させていただいております。

以下から訳となります。

---

NIST Special Publication 800-204A

# サービスメッシュアーキテクチャを用いたセキュアなマイクロサービス型アプリケーションの構築<!-- omit in toc -->

- Ramaswamy Chandramouli, Zack Butcher著
- https://doi.org/10.6028/NIST.SP.800-204A
- 2020年5月

## コンピュータシステム技術に関する報告書<!-- omit in toc -->

国立標準技術研究所（NIST; National Institute of Standards and Technology）の情報技術研究所（ITL; The Information Technology Laboratory）は，国家の測定・標準基盤において技術的リーダーシップを提供することにより米国経済と公共の福祉を促進するものである。ITLは，情報技術の開発と生産的な利用促進のため，テスト，テスト方法，参照データ，PoC実装，技術的分析を開発する。ITLの責任には，連邦政府の情報システムにおいて国家安全保障関連情報以外の情報に対して，費用対効果の高いセキュリティとプライバシーを提供するための管理，運営，技術，物理における標準規格およびガイドラインの開発を含む。Special Publication 800シリーズは，情報システムセキュリティにおけるITLの研究やガイドライン，支援活動，および産業界，政府，学術機関との協力活動を報告するものである。

## 要旨<!-- omit in toc -->

マイクロサービス型アプリケーション構築の傾向が強まったため，そのユニークな特徴から，サービス間の相互作用について，あらゆる側面からのセキュリティ対策が求められている。
マイクロサービスの分散型クロスドメインという性質から，セキュアトークンサービス（STS; Secure Token Service），認証・認可のための鍵管理・暗号化サービス，およびセキュアな通信プロトコルが必要とされている。
（マイクロサービスを構成する）クラスタ化されたコンテナは短命であるため，セキュアなサービスディスカバリが必要である。
同様に，可用性の要件には次の事項が求められる: （a）ロードバランシング，サーキットブレイキング，スロットリングのような回復力技術，（b）（サービス健全性のための）継続的な監視。
**サービスメッシュ**は，個々のマイクロサービスのコードの変更なく効果的に実現されながら，統一的かつ一貫性を持った定義が可能な抽象化レベルでこれら要件の仕様を容易化する最もよく知られたアプローチである。本文書の目的は，マイクロサービス型アプリケーション支援のため，頑強なセキュリティ基盤を形成するプロキシベースのサービスメッシュコンポーネントの構築ガイダンスを提供することである。

## キーワード<!-- omit in toc -->

APIゲートウェイ，アプリケーション プログラミング インタフェース（API），サーキットブレイカー，負荷分散，マイクロサービス，サービスメッシュ，サービスプロキシ

## 注意 特許開示通知<!-- omit in toc -->

情報技術研究所（ITL）は，この出版物のガイダンスまたは要件の準拠に必要となる可能性のある特許クレームの所有者に対し，ITLへの特許クレーム開示を要求する。ただし，特許の保有者はITLの特許請求に応答する義務はなく，ITLはこの出版物に適用される可能性のある特許を特定する調査を行っていない。出版日現在，また本書のガイダンスまたは要件の準拠に必要となる可能性のある特許クレームを特定するための募集を行った時点は，該当した特許クレームはITLに確認されていない。ITLは，本出版物の使用における特許侵害を回避するためにライセンスが必要でないことを表明または暗示するものではない。

## エグゼクティブサマリ

マイクロサービス型アプリケーションのアーキテクチャは，そのスケーラビリティ，デプロイの俊敏性，ツールの可用性の高さから，クラウドや大規模なエンタープライズ環境にてアプリケーションを構築する際の規範となりつつある。
同時に，マイクロサービス型アプリケーションの特性によって，セキュリティ要件は変更・強化が求められている。

この特性およびセキュリティインパクトの例は次のとおりである。

1. マイクロサービスの数が増えるに伴い相互接続が増加し，保護すべき通信リンクも多くなる
2. マイクロサービスは短命であるため，セキュアなサービスディスカバリの仕組みを必要とする
3. 細分化されたマイクロサービスの性質は，きめ細かな認可ポリシーを必要とする

マイクロサービス型アプリケーションを支援するサービス（認証/認可，セキュリティ監視など）は，サービスメッシュのような目的特化のインフラを介して緊密に調整されなければならない。
サービスメッシュのコンポーネントを構築する方法は複数ある。アプリケーション（マイクロサービス）のコードに組み込む方法，ライブラリとして実装してアプリケーションのコードに結合する方法，アプリケーションコードから独立したサービスプロキシとして実装する方法などである。
最後の構築方法は，マイクロサービス型アプリケーションを支援するインフラを構築する多くのシナリオにおいて，スケーラビリティと柔軟性の点で最も効率的であることが分かっている。

本文書の目的は，サービスプロキシ型アプローチにおけるサービスメッシュのコンポーネントの構築ガイダンスを提供することである。
サービスメッシュの構築における推奨事項は以下の通りである。

- サービスプロキシ間の通信設定
- イングレスプロキシの設定
- 外部サービスへのアクセス設定
- IDやアクセス管理の設定
- 監視機能の設定
- ネットワーク回復力技術の設定
- オリジン間リソース共有（CORS; Cross-Origin Resource Sharing）の設定
- 管理操作の権限設定

## 目次<!-- omit in toc -->

- [エグゼクティブサマリ](#エグゼクティブサマリ)
- [1 はじめに](#1-はじめに)
    - [1.1 何故サービスメッシュか](#11-何故サービスメッシュか)
    - [1.2 スコープ](#12-スコープ)
    - [1.3 対象読者](#13-対象読者)
    - [1.4 他NISTガイダンス文書との関係](#14-他nistガイダンス文書との関係)
    - [1.5 本文書の構成](#15-本文書の構成)
- [2 マイクロサービス型アプリケーションの背景とセキュリティ要件](#2-マイクロサービス型アプリケーションの背景とセキュリティ要件)
    - [2.1 認証・認可の要件](#21-認証認可の要件)
    - [2.2 サービスティスカバリ](#22-サービスティスカバリ)
    - [2.3 ネットワーク回復力技術を用いた可用性の改善](#23-ネットワーク回復力技術を用いた可用性の改善)
    - [2.4 アプリケーション監視要件](#24-アプリケーション監視要件)
- [3 サービスメッシュの定義と技術的背景](#3-サービスメッシュの定義と技術的背景)
    - [3.1 サービスメッシュのコンポーネントと機能](#31-サービスメッシュのコンポーネントと機能)
    - [3.1.1 イングレスコントローラ](#311-イングレスコントローラ)
    - [3.1.2 イーグレスコントローラ](#312-イーグレスコントローラ)
    - [3.2 通信ミドルウェアとしてのサービスメッシュ: 何が違うのか](#32-通信ミドルウェアとしてのサービスメッシュ-何が違うのか)
    - [3.3 サービスメッシュ: 最先端環境](#33-サービスメッシュ-最先端環境)
- [4 サービスメッシュの構築推奨事項](#4-サービスメッシュの構築推奨事項)
    - [4.1 サービスプロキシの通信設定](#41-サービスプロキシの通信設定)
    - [4.2 イングレスプロキシの設定](#42-イングレスプロキシの設定)
    - [4.3 外部サービスへのアクセスの設定](#43-外部サービスへのアクセスの設定)
    - [4.4 IDとアクセス管理の設定](#44-idとアクセス管理の設定)
    - [4.5 監視機能の設定](#45-監視機能の設定)
    - [4.6 ネットワーク回復力技術の設定](#46-ネットワーク回復力技術の設定)
    - [4.7 オリジン間リソース共有（CORS）の設定](#47-オリジン間リソース共有corsの設定)
    - [4.8 管理操作の許可設定](#48-管理操作の許可設定)
- [5 概略および結論](#5-概略および結論)
- [参考](#参考)

## 1 はじめに

マイクロサービスアーキテクチャはエンタープライズやクラウドベースのアプリケーションを構築する際の確立された手法である。これには以下の理由がある:

- 俊敏性（agility）: マイクロサービスは疎結合でモジュール性が高いため，他のコンポーネント（マイクロサービス）に影響を与えることなく独立して迅速な修正やデプロイができる
- スケーラビリティ: マイクロサービスの特性から，個々のマイクロサービスを独立にスケールできる
- 利便性（usability）: 明確に定義されたAPI（Application Programming Interface）を使うことにより，多様なマイクロサービスの統合やオンボーディングが容易になる
- ツールの可用性: 自動化ツールの可用性が向上するため，エラーのない設定やデプロイが容易になる

上記の利点にもかかわらず，マイクロサービス型アプリケーションのアーキテクチャはセキュリティ要件の修正や強化を伴う以下の課題がある。

- マイクロサービスの数が増えるに伴い相互接続が増加し，保護すべき通信リンクも多くなる
- コンポーネント（マイクロサービス）が動的に生成・削除されるため，セキュアなサービスディスカバリが必要となる
- ネットワークの境界線という概念がない
- あらゆるマイクロサービスを信頼できないものとして扱わなければならない
- マイクロサービスのきめ細かな性質は，マイクロサービスごとの細かな認証を必要とする。しかし，そのためにはセキュリティポリシーを一元的に定義し，その設定を個々のマイクロサービスに反映させることで，マイクロサービス全体に渡った均一かつ一貫したセキュリティポリシーの施行を可能とさせる必要があるかもしれない

### 1.1 何故サービスメッシュか

上述したマイクロサービス型アプリケーションのセキュリティ要件のためにアプリケーションを支援するインフラと，そのインフラに関連するサービス（例えばセキュリティ）は，緊密に連携されていることが望ましい。
サービスメッシュはそのような目的に特化したインフラの一つである。
サービスメッシュを実現するコードは，マイクロサービス型アプリケーション・アーキテクチャのコンポーネントに対し，以下のように体系付けられる（各アーキテクチャパターンは，SM-ARxと表記する。SMはサービスメッシュ，ARはアーキテクチャ，xはシーケンス番号である）。

- SM-AR1: サービスメッシュのコードはマイクロサービスアプリケーションのコードに組み込まれ，アプリケーション開発フレームワークの不可欠な要素となる。
- SM-AR2: サービスメッシュのコードはライブラリとして実装され，アプリケーションはAPI呼び出しを介してサービスメッシュの機能に結合される。
- SM-AR3: サービスメッシュの機能はプロキシ内に実装されている。各プロキシはマイクロサービスインスタンス（訳者注:コンテナなど個々のマイクロサービスを構成する実態）の前段に配置され，マイクロサービス型アプリケーションにインフラサービスをまとめて提供する。このプロキシは「サイドカープロキシ」と呼ばれ，アプリケーションコードから独立した実装・運用が可能である。サイドカープロキシは種々のプラットフォーム（様々なプログラミング言語やアプリケーション開発フレームワーク）にとっての最小公分母なAPI―――つまり，ネットワークを用いることにより，種々のプラットフォームの一貫した制御を可能としている。
- SM-AR4: サービスメッシュの機能はプロキシに実装されるが，SM-AR3のようなマイクロサービスインスタンスごとではなく，ノード（物理ホスト）ごと配置される

### 1.2 スコープ

本文書の目的のため，サービスメッシュのアーキテクチャはSM-AR3の構成のみを検討する。
SM-AR3はアプリケーションのサービスコードを修正することなく，マイクロサービス型アプリケーションにあらゆるセキュリティ機能を提供する目的特化のインフラ層となる。
SM-AR4と比べ，SM-AR3は特権のエスカレーション範囲やノイジーネイバー問題（訳者注:高負荷なインスタンスが周辺のインスタンスの性能に影響を及ぼすこと）が少ない。これはSM-AR3がマイクロサービスインスタンスごとに一つのサービスプロキシのインスタンスを配置しており，アプリケーションのトラフィックは確実にサービスプロキシによってのみ中継されるためである。
SM-AR1およびSM-AR2と比較して，SM-AR3はサービスメッシュの機能を提供するモジュールをアプリケーションのライフサイクルから分離できるため，SM-AR2で発生するような複数言語を跨いだ複数バージョンのライブラリを保守することによる組合せ爆発問題を回避できる。
この文脈に基づき，本文書の観点におけるサービスメッシュの主機能は，マイクロサービスのコードと密結合しないエージェントや機能モジュールが，クライアントからマイクロサービス，およびマイクロサービスからマイクロサービスへの通信を調停（mediate）および仲介（broker）することである。

### 1.3 対象読者

サービスメッシュフレームワークを用いてマイクロサービス型アプリケーションを支援するための本ガイダンス文書の対象読者は，マイクロサービス型アプリケーションのためのセキュリティフレームワークを設計したいセキュリティソリューション設計者や，企業やクラウドに存在する様々なマイクロサービス型アプリケーションのための共通インフラサービス・フレームワークを構築するシステムインテグレータ等である。

### 1.4 他NISTガイダンス文書との関係

本ガイダンス文書は，マイクロサービス型アプリケーションのための特定セキュリティフレームワークやインフラストラクチャの構築に焦点を当てている。
マイクロサービス型アプリケーションの特性と，その全体的なセキュリティ要件や戦略の理解することは有益であり，その情報はNIST Special Publication(SP) 800-204「[マイクロサービス型アプリケーションシステムのセキュリティ戦略\[1\]][1]」にて提供される。

### 1.5 本文書の構成

本文書の構成は以下の通りである。

- 2章では，[\[1\]][1]で述べたマイクロサービス型アプリケーションのセキュリティ要件を参照し，マイクロサービス型アプリケーションのセキュリティ要件を再確認する。
- 3章では，サービスメッシュを紹介し，コンポーネント，機能，マイクロサービス型アプリケーションにおける通信ミドルウェアとしての独自の役割の概説を述べる。
- 4章では，サービスプロキシ，イングレスプロキシ，イグレスプロキシ，IDとアクセス管理，監視機能，ネットワーク回復力の技術，オリジン間リソース共有（CORS）等の構成領域にまたがるサービスメッシュコンポーネントの詳細な構築推奨事項を述べる。
- 5章では，まとめや結論を述べる。

## 2 マイクロサービス型アプリケーションの背景とセキュリティ要件

マイクロサービス型アプリケーションの定義と詳細，脅威およびその対策となるセキュリティ戦略はNIST SP 800-204，[マイクロサービス型アプリケーションシステムのセキュリティ戦略\[1\]][1]にて述べられている。
本章の目的はこの種のアプリケーションのセキュリティ要件を再確認して磨きをかけ，第3章で述べるサービスメッシュの機能において，これら要件が如何に満たされるか背景を提供することである。
これにより，第4章の要件を満たすサービスメッシュコンポーネントのデプロイ推奨事項の開発が容易になる。

### 2.1 認証・認可の要件

認証およびアクセスポリシーは，マイクロサービスが公開するAPIの種類によって異なる場合がある。これにはパブリックAPI，プライベートAPI，パートナーAPI（ビジネスパートナーのみ利用可能なAPI）などがある。マイクロサービスは複数存在し，認証ポリシーはそれらすべてをカバーするように定義される必要がある。更に証明書ベースの認証は，証明書の生成・管理および鍵管理のために公開鍵基盤（PKI; Public Key Ingrastracture）を必要とする。その上，全てのマイクロサービスのリソースをカバーする認可モジュールを構築して全てのサービスリクエストにきめ細かな認可を提供しなければならない。

### 2.2 サービスティスカバリ

レガシーな分散システムの場合，複数あるサービスは指定されたロケーション（IPアドレスおよびポート番号）で稼働するよう設定されている。
マイクロサービス型アプリケーションには以下のシナリオが存在するため，堅牢なサービスディスカバリの仕組みが求められる。

1. 無数のサービスが存在し，各サービスは動的にロケーションが変化する多数のインスタンスを有する。
2. 各マイクロサービスは仮想マシン（VM; Virtual Machine）やコンテナとして実装され，IPアドレスが動的に割り当てられる場合がある。特にIaaS（Infrastructure as a Service）やSaaS（Software as a Service）のようなクラウドサービスで稼働する場合，これは顕著である。
3. オートスケーリングなどの機能により，サービスに紐付けられたインスタンスの数が負荷に応じて変化する

以上の特徴から，サービスリクエストを行いながらサービスを検出する機能は必須の要件となる。
本機能を実現する場合，サービスレジストリを利用することが一般的である。
サービスレジストリとは，マイクロサービス型アプリケーションのために作成されたサービスインスタンスを登録し，オフラインとなったサービスインスタンスを削除するディレクトリで構成される。

### 2.3 ネットワーク回復力技術を用いた可用性の改善

- 負荷分散: 過負荷による応答遅延や障害を避けるため，サービスに複数インスタンスを持たせ，これらインスタンスへ均等に負荷を分散される必要がある。
- サーキットブレイカー: 巨大な分散システムは，どのように設計したとしても一つの特徴を持つ―――つまり，小規模・局所的な障害がシステム全体の壊滅的な障害に進展しやすい。サービスメッシュは下層システムに限界がきたとき，負荷を弾き，迅速に障害を発生させることで，このような障害の進展から保護するよう設計されなくてはならない。サーキットブレイキングとは，マイクロサービス・インスタンスからの応答の失敗回数にしきい値を設け，失敗がしきい値を超えたときに，（電子回路の遮断器を作動させたように）そのインスタンスへのリクエストを遮断する設定を意味する。これは連鎖的な障害の可能性を緩和し，ログ分析や修正の実装，障害が発生中のインスタンスを更新する時間を稼ぐ。このように，サーキットブレイキングは，サービスリクエストへの応答が完全に途絶えることを防ぐ一時的な措置である。サービスが応答可能になれば，サービスリクエストは再びインスタンスに送られる。
- 速度制限（スロットリング）: すべてのクライアントがサービスを継続的に利用可能とするために，マイクロサービスに入るリクエストの速度を制限する必要がある。
- ブルーグリーン・デプロイメント: 新バージョンのマイクロサービスがデプロイされたとき，旧バージョンを使用しているクライアントのリクエストを新バージョンにリダイレクトする。これには両バージョンのロケーションを持続して認知するようプログラムされたAPIゲートウェイが使用される。
- カナリアリリース: あらゆる稼働シナリオの下でレスポンスや性能メトリクスの正確性を完全に知ることはできないため，新バージョンのマイクロサービスにはまず限られた量の通信のみを送信する。その動作特性について十分なデータが得た後に，全リクエストを新バージョンのマイクロサービスに送信する。

### 2.4 アプリケーション監視要件

攻撃の検知や（可用性に影響を与える恐れのある）サービスの劣化要因を特定するため，分散ロギング，メトリクスの生成，性能分析，トレーシングなどを通じて，マイクロサービスに出入りするネットワーク通信を監視する必要がある。

## 3 サービスメッシュの定義と技術的背景

前章でのマイクロサービスの説明から，マイクロサービスには大きく分けて2つの機能があると分かる[\[2\]][2]。

- **ビジネスロジック**: ビジネス機能，計算，サービス構成・統合ロジックを実現する
- **ネットワーク機能**: サービス間通信の仕組みを担う（所定のプロトコルを介した基本的なサービス呼び出し，回復力や安定性の図るデザインパターンの適用，サービスディスカバリ等）。これらのネットワーク機能はOSレベルのネットワークスタックの上に構築されている。

ビジネスロジック機能はビジネス処理を実行もしくは支援するサービスであるため，マイクロサービスのコードにおいて不可欠な部分である。
このマイクロサービスがネットワーク機能を直接実行すると，採用したプログラミング言語や開発フレームワークによって異なるライブラリを使用する点で難しい。
実際のところ，開発やランタイムプロセス最適化のため，マイクロサービスは同一アプリケーション内であっても複数の言語（Java，JavaScript，Pythonなど）を用いているため，サービスノードごとに通信能力を提供することは面倒な作業となる。

サービスメッシュはサービス間通信を円滑化する一連のインフラ機能を備えた目的特化のインフラ層であり，その機能にはサービスディスカバリ，ルーティング，内部負荷分散，トラフィック設定，暗号化，認証，認可，メトリクス，および監視がある。
これは，サービスインスタンスの生成・削除や継続的な再配置によりネットワークトポロジが変化する環境において，ポリシーを介してネットワークの振る舞い，マイクロサービスのインスタンスID，通信フローの宣言的定義を行う機能を提供する。
サービスメッシュはOSI（Open Systems Interconnection）参照モデル（例えば，TCP/IP;Transmission Control Protocol/Internet Protocol）におけるトランスポート層上部の抽象化層に位置し，サービスが持つセッション層（OSI参照モデルの第5層）の懸念事項に対処するためのネットワーキングモデルと見なすことができる。
ただし，ビジネスロジックを完全に把握しているのはマイクロサービスレベルだけであるため，依然としてマイクロサービスレベルで詳細な認証を行う必要が出てくる場合もある。

また，サービスメッシュは「アプリケーションサービス間の通信を最適化する分散コンピューティングのミドルウェア[\[3\]][3]」とも定義できる。
サービス間通信はプロキシを用いることで最も効果的に有効となる（[1.1節](#11-何故サービスメッシュか)参照）。
サービスメッシュは通常，アプリケーションに気付かれることなく，アプリケーションコードの横にデプロイされる軽量ネットワークプロキシ群として実現される[\[4\]][4]。

加えて，サービスメッシュは監視やセキュアな通信にも活用できる。
サービスメッシュはクラスタの全トラフィックを傍受してルーティングを行う他，ヘルスメトリクスも収集しているため，トラフィックを把握（learn）し，インテリジェントにルーティングできる。
この高度な機能の例には，A/Bテスト，カナリアデプロイメント，ベータ版リリース（beta channel），自動リトライ，サーキットブレイカー，障害の注入（injecting faults）などがある。
これらの機能はサービスメッシュがクラスタのトラフィック全体を見て把握できるからこそ可能となる。

アプリケーションを構成するマイクロサービスの数が数百，数千のオーダーのとき，サービスメッシュを導入することは経済的であると考えられる。しかし，サービスメッシュに欠点がないわけではない。
マイクロサービスごとにサービスプロキシが必要になるため，ランタイムインスタンスの数が増え，アプリケーション全体の攻撃面（attack surface）は増加する。
サービスプロキシの組込み機能が増えると，通信のボトルネックにもなりうる。

### 3.1 サービスメッシュのコンポーネントと機能

サービスメッシュは2つの主要なアーキテクチャ層もしくはコンポーネントから構成される:

- データプレーン
- コントロールプレーン

データプレーンはサービスメッシュ内にて相互接続されたプロキシ群を指し，サービス間通信を制御する。
データプレーンはデータパスであり，アプリケーションからのリクエストを転送する機能を持つ。
またデータプレーンは，ヘルスチェック，負荷分散，サーキットブレイキング，タイムアウト，リトライ，認証，認可などの高度な機能も提供する[\[5\]][5]。
この特別なプロキシはサービスインスタンスごとに作成され（すなわち，サイドカープロキシ），セキュリティ（アクセス制御などの通信関連）を強制するために必要なランタイム操作を行う。これは，コントロールプレーンから本プロキシにポリシー（アクセス制御ポリシーなど）を注入することによって可能となる。

コントロールプレーンはメッシュ全体のデータプレーン（プロキシ）の動作を制御・設定するためのAPI群およびツールである。
サービスメッシュのコントロールプレーンはオーケストレータのコントロールプレーンとは異なる。前者はサービスメッシュを制御し，後者はクラスタを制御する。
コントロールプレーンは，ユーザの認証ポリシーや名前情報を指定し，（一般的なテレメトリ収集の）メトリクスを集め，データプレーン全体を設定する場所である[\[6\]][6]。
すべてのセキュリティ機能に必要な知能，データ，およびその他人工物はコントロールプレーンにある。
コントロールプレーンには，認証証明書を生成するソフトウェアとそれを保持するリポジトリ，認証ポリシー群，認可エンジン，各マイクロサービスに関するテレメトリ/モニタリングデータを受信・集約するソフトウェア，負荷分散・サーキットブレイキング・流量制御などネットワークの挙動を変更する様々な機能のAPIを持つ。
サービスメッシュプラットフォームのコントロールプレーンは，マイクロサービス型アプリケーションのオーケストレーションプラットフォームと（サービスレジストリなどの重要データを取得するために）統合されなければならず，使いやすい統合機能を持つべきである。
コントロールプレーンはサービスメッシュの重要コンポーネントであるため，高可用性や分散配置が求められる。
コントロールプレーンは，設定ファイル，API呼び出し，ユーザインタフェースを通して操作できる[\[7\]][7]。

通信を提供する処理の一環として，以下の機能がサポートされる[\[1\]][1][\[2\]][2]:

- 認証・認可: 証明書の生成，鍵管理，ホワイトリストやブラックリスト，シングルサインオン（SSO）トークン，APIキー
- セキュアなサービスディスカバリ: 相互TLS（Transport Layer Security），暗号化，動的な通信経路生成，複数プロトコルのサポート，必要に応じたプロトコル変換（HTTP（Hypertext Transfer Protocol）/1.xやHTTP/2，gRPCなど）
- 通信のための回復力や安定化機能: サーキットブレイカー，リトライ，タイムアウト，障害の注入・ハンドリング，負荷分散，フェイルオーバー，流量制御，リクエスト複製
- 可観測性/監視機能: ロギング，メトリクス収集，分散トレーシング

### 3.1.1 イングレスコントローラ

サービスメッシュのサービスプロキシはイングレストラフィック（サービス間通信ではなく，外部からマイクロサービスアプリケーションに流入するトラフィック）の制御に利用できる。
この意味で，これはAPIゲートウェイの機能を実現する。
概念的に，イングレスコントローラは外部クライアントのサイドカープロキシとみなせる。
イングレスコントローラ（フロントプロキシとも呼ばれる）は以下の機能を提供する:

- サービスメッシュ内の実際のAPIの盾となる全クライアント共通のAPI。
- Webフレンドリーなプロトコル（HTTP/HTTPS（Hypertext Transfer Protocol Secure）など)からのマイクロサービスで使われるプロトコル（RPC/gRPC/REST(Representational State Transfer）など)への変換。
- クライアントの単一呼出をサービスメッシュ内の複数サービス呼出として受信した結果の合成。
- 負荷分散
- パブリックなTLSの終端

### 3.1.2 イーグレスコントローラ

サービスメッシュのサービスプロキシはイーグレストラフィック（メッシュ外のマイクロサービスを宛先とするマイクロサービスの内部トラフィック）の制御に利用できる。
この意味で，これは流出専用のゲートウェイとして機能する。
概念的に，イーグレスコントローラは一つ以上の外部サーバのサイドカープロキシとみなせる。
イーグレスコントローラは以下の機能を提供する:

- 外部ネットワークとの通信のためにホワイトリストに登録する単一のワークロード（ホスト，IPアドレスなど）。例えば，ファイアウォールにイーグレスプロキシを許可するだけで，ローカルネットワークからのトラフィックを転送できる。
- 認証情報(credential)の変換: アプリケーションが外部システムの認証情報に直接アクセスすることなく，内部（メッシュ）のID認証情報から外部認証情報（SSOトークンやAPIキーなど）に変換する。
- マイクロサービス向けプロトコル（RPC/gRPC/RESTなど）からウェブ向けプロトコル（HTTP/HTTPSなど）へのプロトコル変換。

### 3.2 通信ミドルウェアとしてのサービスメッシュ: 何が違うのか

サービスメッシュ以前として，インフラ機能（マイクロサービス型アプリケーションのような分散システムのためのサービスディスカバリ，負荷分散，サーキットブレイキング，障害の注入，セキュリティ監視，認可，分散トレーシングなど）を提供するためには，一連のコンポーネントとフレームワークを慎重に選択されなければならない。
コンポーネントの中には特定のフレームワークでしか機能しないものもあり，フレームワーク自体が特定の言語に縛られている場合もある。
加えて，アプリケーションサービスのコードは，これらのコンポーネントで動作するように，あるいはこれらのコンポーネントに統合されるように修正されなければならない[\[8\]][8]。

サービスメッシュの場合，動作はネットワークレベルで行われるため，個々のマイクロサービスを構成する技術やプログラミング言語は問題にならない。
例えば，HTTPサーバーを開発するとき，マイクロサービスアプリケーションの開発者はJava，C++，Rust，Go，JavaScript，Pythonなどの言語を自由に選択できる。
サービスメッシュは，アプリケーションコードをサービス間通信の管理から分離する。
アプリケーションコードは，ネットワークトポロジー，サービスディスカバリ，ロードバランシング，接続管理ロジックを知らなくてよい[\[9\]][9]。
同様に，テレメトリ，トラフィックシェーピング，サービスディスカバリ，ネットワークポリシー制御などの機能も追加設定なしで使用できる。

サービスメッシュは通信ミドルウェアとして定義されることから，次は「他の分散システムミドルウェアとどう違うのか？」という疑問が出てくる。
分散システムのための伝統的なミドルウェアにはアプリケーション配信コントローラ（ADCs; Application Delivery Controllers），ロードバランサ，APIゲートウェイなどがある。
これらのミドルウェア機器は，重いコストと運用のオーバヘッドに加えて，アプリケーションコンポーネントが疎結合のモジュール型マイクロサービスの形をしている場合には不向きである。なぜなら，これらのコンポーネントはモノリシックなアプリケーションでは不要な動的ディスカバリなどのきめ細やかな能力や機能を必要とするためである。

分散システムにおける従来の通信ミドルウェアコンポーネントの不向きさ，及びマイクロサービス型アプリケーションシステムにおける軽量な解決方法の必要性を理解するためには，それらシステムにおける通信の性質を考察せよ。
様々なタイプのクライアントが，膨大な数のマイクロサービスで構成されたアプリケーションと接続している。
クライアントとアプリケーションサービス間の通信トラフィックは南北（north-south）トラフィックと呼ばれ，マイクロサービス間の通信は東西（east-west）トラフィックと呼ばれている。
マイクロサービス型アプリケーションは，モノリシックなアプリケーションに比べ，コンポーネントとしてのマイクロサービスの数が多いため，東西のトラフィックの量が非常に多い。本番アプリケーションに許容できるレベルの性能を提供するには，サービスメッシュのような軽量な通信ミドルウェアが必要となる。

マイクロサービス型アプリケーションはコンテナやそのオーケストレーションサービスなしでも実装可能ではあるが，クラウドネイティブなアプリケーション（サービスベースのアーキテクチャ，API駆動通信，コンテナベースのインフラストラクチャ，DevOps（DevelopmentとOperations）プロセス（例えば，継続的インテグレーション，アジャイル開発，継続的デリバリ，開発者・品質保証チーム・セキュリティ専門家・IT運用者・ビジネス利害関係者による共同開発[\[3\]][3]）を備える）と見なされやすい。
この見方の理由の一部は，モノリシックなソフトウェア開発とデプロイが，APIベースの通信による疎結合なサービスベースのアーキテクチャではなく，緊密に統合されたアプリケーションモジュールを備えたサーバー中心のインフラストラクチャに依存していることにある。

### 3.3 サービスメッシュ: 最先端環境

概念的には，サービスメッシュを用いると数十のインスタンスからなるサービスが数百あるようなマイクロサービスアーキテクチャに基づくあらゆるアプリケーションのためのインフラサービスを提供できる。
しかし，今日利用可能な既製品の実装に基づくと，サービスメッシュは以下のような構成のアプリケーションプラットフォームに最も適しており，生産性が高めることができる。

- 各マイクロサービスはコンテナで実装されている。
- アプリケーションはコンテナのオーケストレーションツールを使用して管理されるコンテナクラスタを（可用性および性能向上のために）使用する。
- アプリケーションは，クラウドプロバイダが提供するCaaS（Container-as-a-Service）にホストされており，コンテナ管理/オーケストレーション環境で必要なデプロイおよび設定のツールを備える。

## 4 サービスメッシュの構築推奨事項

本章はサービスメッシュの構築オプションを検討し，様々なアプリケーションシナリオに対応したセキュアなマイクロサービス型アプリケーションのための推奨事項を提供する。
主要なランタイム機能はプロキシによって実行されるため，最初の構築推奨事項はプロキシ機能を様々な側面から設定する文脈で行われる。
各構築推奨事項は，SM-DRxという記号で識別される。SMはサービスメッシュ，DRはデプロイ推奨事項（Deployment Recommendation），xはシーケンスの番号を表す。

### 4.1 サービスプロキシの通信設定

**サービスプロキシへの許可トラフィックに関する推奨事項（SM-DR1）**: サービスプロキシは，関連サービスが受付可能なトラフィックのプロトコル及びポートを指定する機能を有するべきである。
デフォルトでサービスプロキシは指定されたトラフィック以外通すべきではない。

**サービスプロキシのアクセス可能性に関する推奨事項（SM-DR2）**: サービスプロキシがアクセスできるサービスの組は制限されなければならない。
サービスプロキシはNamespaceやNamespace内の特定の名前付きサービス，またはサービスのランタイムIDに基づいてアクセスを制限する機能を有するべきである。
サービスメッシュのコントロールプレーンに対しては，ディスカバリ，異なるポリシー，およびテレメトリデータ中継のため，常にアクセスできなければならない。

**プロトコル変換機能に関する推奨事項（SM-DR3）**: サービスプロキシは，対象のマイクロサービスとは異なるプロトコルで通信するクライアントをサポートする機能を有するべきである（例えば，REST/HTTPリクエストからgRPCリクエストへの変換やHTTP/1.1からHTTP/2へのアップグレードなど）。
これは，クライアントプロトコルごとに別個のサーバを構築ことによる攻撃面の増加を回避するために必要となる。

**ユーザ拡張性に関する推奨事項（SM-DR4）**: サービスプロキシは，ネットワーク機能の制御のため，組み込みロジックに加えてカスタムロジックを定義する機能を有するべきである。
これは，ユースケース固有のポリシー（既存や自作のポリシーエンジンなど）実装に向け，サービスプロキシを拡張可能にするために求められる。
この実装は，サンドボックス化や，使用言語のAPI/ランタイム制限，安全性確保のための事前解析の実行など，拡張性のリスクを制御する手段を提供する必要がある（例えば，WASM; Web AssemblyやeBPF; extended Berkeley Packet Filter）。

**プロキシへの動的設定機能に関する推奨事項（SM-DR5）**: 静的設定に加え，動的にプロキシを設定するオプション（イベント駆動型の設定更新など）を持つべきである。
言い換えると，デプロイ時に既知ではなく動的であることが期待されるエンティティのためのディスカバリサービスがあるべきである。
更にプロキシは，以前の設定下における未解決リクエストを効率的に処理（すなわち，完了または終了）しながら，実行時に新しい動的設定へとアトミックにスワップするべきである。
これはユーザトラフィックの不具合やダウンタイムなしに（つまり，サービスプロキシを再起動することなく）実行時にタイムリーなポリシー変更を行うために必要となる。

**アプリケーションサービスからそのプロキシへの設定通信に関する推奨事項（SM-DR6）**: アプリケーションサービスとそのプロキシはループバックアドレス（localhostのIPアドレスやUNIXドメインソケットなど）のみで通信すべきである。
更に，サービスプロキシ間では互いに全てのデータパケットが暗号化される相互TLS（mTLS）セッションによってのみ通信されるべきである。

### 4.2 イングレスプロキシの設定

**イングレスプロキシに関する推奨事項（SM-DR7）**: サービスプロキシと同様に（スタンドアロンの）イングレスプロキシはトラフィックルーティングのルール設定を行う機能を有するべきである。
これは，アプリケーションデプロイの端点まで一貫したルーティングポリシーを適用する必要があるためである。


### 4.3 外部サービスへのアクセスの設定

マイクロサービス型アプリケーションの一部サービスは，パブリックもしくはプライベートのWeb APIやサードパーティのサービス，レガシーアプリケーション，VMや（サービスメッシュが動作するクラスタとは異なる）別クラスタのような異なる仮想化インフラのアプリケーションにアクセスしなければならない場合がある。
これらリソースへのアクセスに同等のセキュリティ保証を提供するために，次の推奨事項が提供されている。

**外部リソースのアクセス制限に関する推奨事項（SM-DR8）**: メッシュ外の外部リソースやサービスへのアクセスはデフォルトで無効にし，指定された宛先へアクセスを制限する明示的なポリシーによってのみ許可されるべきである。
加えて，これらの外部リソースやサービスは，サービスメッシュ自体のサービスとしてモデル化されるべきである（例えば，サービスメッシュのサービスディスカバリに含めるなど）。

**外部リソースへのセキュアなアクセスに関する推奨事項（SM-DR9）**: サービスメッシュ内のサービスに設定されているものと同等の可用性向上機能（リトライ制御，タイムアウト，サーキットブレーカなど）を外部のリソースやサービスへのアクセスにも提供しなければならない。

**イーグレスプロキシに関する推奨事項（SM-DR10）**: （スタンドアロンの）イーグレスプロキシも，サービスプロキシやイングレスプロキシと同様，トラフィックルーティングルールを設定する機能を有するべきである。
デプロイされた場合，外部リソースやサービスへのアクセスは，これらのイーグレスプロキシによって仲介されるべきである。
イーグレスプロキシは，アクセスポリシーと可用性ポリシー（SM-DR8）が実装されるべきである。
これは，従来のネットワーク指向のセキュリティモデルで動作する場合に有効である。
例えば，インターネットへの外向きトラフィックがネットワーク内の特定のIPからのみ許可されている場合，イーグレスプロキシはメッシュ内の様々なサービスのトラフィックをプロキシしながらそのIPアドレスで動作するように設定できる。


### 4.4 IDとアクセス管理の設定

マイクロサービス型アプリケーションの主な通信相手は，クライアント，マイクロサービス，外部サービスの3種類ある。
いかなるペア（クライアントからマイクロサービス間，マイクロサービス同士，マイクロサービスから外部サービス）の通信イベントの間，両エンティティには異なるIDを持たせ，相互認証を行う必要がある。
相互TLS（mTLS）はこれを行うためのデファクトの仕組みであるため，クライアントまたはマイクロサービスが保持する認証証明書には「subject name」または「subject alternative name」フィールドにそのIDを含んでいる必要がある。
このIDは，サーバID（ホストまたはドメインとしても知られる）またはサービスID（通常はサービスアカウントID）のいずれかである。
証明書のデプロイに関する推奨事項は以下の通りである。


**ユニバーサルIDドメインに関する推奨事項（SM-DR11）**: マイクロサービスのあらゆるインスタンスのIDは一貫性と一意性を有するべきである。
一貫性としてサービスはどこで動作しているかによらず同じ名前を持つべきであり，一意性としてシステム全体を通してサービス名はそのサービスにのみ対応すべきである。
これは，ロケーションによって論理サービスが異なるという意味ではない。各サービスに独自のDNS（Domain Name System）名を割り当てるような典型的なDNSの使い方は，この推奨事項を満たしている。
システムポリシーを管理可能にするため，サービスの一貫した名前（ID）が必要である。

**証明書への署名に関する推奨事項（SM-DR12）**: サービスメッシュのコントロールプレーンが持つ証明書管理システムは自己署名証明書の生成機能を無効にすべきである。
この機能は，サービスメッシュ内の他の全てのID証明書のために初期署名証明書を生成するために頻繁に使用される。
代わりに，メッシュのコントロールプレーンで使用される署名証明書は，常に信頼された企業の既存PKI（Public Key Ingrastructure;公開鍵基盤）のルートに根付いたものとし，起動時にサービスメッシュのコントロールプレーンにセキュアに提供されるべきである。
これにより，既存のPKIを用いたこれらの証明書の管理（失効処理や監査など）がシンプルになる。
更に，ローテーションを簡素化し，きめ細かな失効を可能にするために，異なるドメインに対して別々の中間署名証明書を生成することが推奨される。

**ID証明書のローテーションに関する推奨事項（SM-DR13）**: マイクロサービスのID証明書の有効期間は，インフラ内で管理できる範囲で短くすべきである（可能なら数時間のオーダー）。
攻撃者は資格情報の有効期限が切れるまでしか資格情報を利用したサービスのなりすましができず，資格情報の再盗用は攻撃の難易度を上げるため，攻撃を制限することに役立つ。

**ID変更時に接続を循環させるサービスプロキシに関する推奨事項（SM-DR14）**: サービスプロキシのID証明書を更新するとき，サービスプロキシは既存の接続を効率的に廃棄し，すべての新しい接続を新規の証明書で確立すべきである。
証明書はmTLSのハンドシェイク時のみ検証されるため，新しい証明書の発行時に既存の接続を置き換えることは厳密には必要ない。
むしろ，これは攻撃を一定時間内に制限するために重要である。

**非署名型識別証明書に関する推奨事項（SM-DR15）**: マイクロサービスのIDに使用される証明書は，署名証明書であってはならない。

**証明書発行前のワークロード認証に関する推奨事項（SM-DR16）**: サービスメッシュのコントロールプレーンが持つ証明書管理システムは，サービスインスタンスにID証明書を発行する前にサービスインスタンスを認証すべきである。
多くのシステムでは，システムのオーケストレーションエンジンに対してインスタンスを裏付けし，他のローカル証明（例 HSM; Hardware Security Moduleから取得したシークレットなど）を使用することで実現できる。
同様に，サービスメッシュのコントロールプレーンが持つ証明書管理システムの署名証明書をプロビジョニングする際も注意すべきである。
この署名証明書はサービスメッシュのコントロールプレーンによってのみ取得可能で，かつ何らかの形式で裏付けが行われた後にのみ取得可能とされるべきである。

**セキュアなネーミングサービスに関する推奨事項（SM-DR17）**: mTLSが用いる証明書がサーバIDを運ぶ場合，サービスメッシュはサーバIDをセキュアディスカバリサービスにより提供されるマイクロサービスの名前にマッピングするようなセキュアなネイミングサービスを提供すべきである。
この要件はサーバをマイクロサービスにとって認可された場所にあることを確かめ，ネットワークのハイジャックから保護するために必要である。

mTLSで用いる証明書がサービスIDを運ぶ場合，追加のセキュアなネーミングサービスは不要である。
これは同様にマイクロサービスを異なるネットワークドメイン（異なるクラスタやクラウド）に移植したときでも，新しい環境に合わせてID，および関連付けられたアクセス制御ポリシーを定義し直す必要がなくなる。

サービスIDに基づくマイクロサービスの証明書を設定することで，通信する2つのサービスがセキュアな通信チャネルを確立できる。しかし，その場所での通信が全て許可サれているかは指定されていない。
これを指定するには，マイクロサービスノードごとに流入および流出トラフィックの許可ポリシーを定義する機能が必要となる。

**きめ細かいIDに関する推奨事項（SM-DR18）**: マイクロサービスごとに独自のIDを持ち，このマイクロサービスのすべてのインスタンスは，実行時に同じIDを提示すべきである。
これにより，与えられたネームスペースにおいて，マイクロサービスのレベルでのアクセスポリシーが可能となる。
一般的なマイクロサービスはデフォルトで，サービスごとではなくネームスペースごとのIDを発行するため，特に指定がない限り，同じネームスペース内のすべてのサービスが同じ実行時IDを提示してしまう。このため，本機能が必要となる。
更に，ラベルを使用してサービス名（ID）を拡張することにより，きめ細かなロギング設定や，認可ポリシーをサポートできる。

**認証ポリシーのスコープに関する推奨事項（SM-DR19）**: 認証ポリシーのスコープを指定する機能は，以下の最小オプションを持つべきである。
(a)全てのネームスペースにおける全てのマイクロサービス，(b)特定のネームスペースにおける全てのマイクロサービス，(c)特定のネームスペースにおける特定のマイクロサービス（SM-DR17の実行時IDを使用）。

上で述べたアプローチは静的なパラメータ（サービスIDや事前定義ポリシー）を用いた認証を可能にしている。
シナリオによってはコンテキスト情報（マイクロサービスを呼び出しているユーザなど）を組み込む機能という追加要件がある。
この仕組にはプラットフォームに依存しないフォーマット（JWT; JavaScript Object Notation（JSON） Web Tokenなど）でエンコードされたトークンが使われる。

**認証トークンに関する推奨事項（SM-DR20）**: トークンはデジタル署名および暗号化されるされるべきである。これは，トークンに含まれるクレームがアクセス制御を行う認証IDの一部またはその拡張として使われるため，信憑性の保証が必要だからである。
更に，これらのトークンは（ネットワークパスが関与しないことを確実にするために）ループバックデバイス，または暗号化されたチャネルを通してのみ渡される必要がある。

### 4.5 監視機能の設定

全てのプロキシ（イングレス，イーグレス，サービス）は全ての監視データを収集する機能を有するべきである。
監視データはPeter Bourgonによって特定されたように，ソフトウェアシステムにおけるロギング，メトリクス，トレーシングの3種のカテゴリに分類される[\[10\]][10]。
この機能はサービスメッシュが有する上記データカテゴリの一つ以上を生成できる専用ツールの統合サポートを有効にすることで実現できる。
例えば，イベントロギングのためのAppSensorやFluentd，集約可能なメトリクスのためのPrometheus，分散トレーシングのためのJaeger[\[11\]][11]やZipkin[\[12\]][12]などがある。
これらはサービスメッシュの導入チームがデータ収集パイプラインの構築にかける労力を最小限に抑え，データ分析へ注力させることができる。
ユースケースの文脈に応じ，この例にはイベントマイニングや異常検知，サービス依存性抽出などもある。

ロギングとしては，最低限，一般的な攻撃を検出できるよう，下記イベントを記録すべきである。

**イベントのログ記録に関する推奨事項（SM-DR21）**: プロキシは入力検証(validation)エラーと(意図しない)余計なパラメータのエラー，クラッシュ，コアダンプのログを取るべきである。一般的な攻撃検出機能は署名なし（Bearer）トークンのリプレイ攻撃やインジェクション攻撃を検出可能とすべきである。

**リクエストのログ記録に関する推奨事項（SM-DR22）**: プロキシはイレギュラーなリクエスト（例えば，HTTPでステータスコードが200以外のレスポンス）のために，少なくともCommon Log Format（訳者注: リモートホスト，リモートログ名，リモートユーザ，受付時刻，リクエストメソッド，ステータスコード，レスポンスサイズ）のフィールドのログを取るべきである。
メトリクスが利用可能な場合，成功したリクエストのログ（例えば ステータスコードが200のレスポンス）にはほとんど価値がない傾向にある。

**ログメッセージの内容に関する推奨事項（SM-DR22※1）**: 
ログメッセージには少なくとも，イベント日時，マイクロサービス名もしくはID，リクエストトレースID，メッセージ，その他コンテキスト情報（リクエストしたユーザのIDやURL）を含めるべきである。
ただし，Bearerトークンのような機密情報はマスクするよう注意すべきである。

※1 訳者注:原文にてSM-DR22が重複しているため踏襲

メトリクスとしては，ビジネスロジック決定の結果や接続の試行，その他挙動について，正常で妥協のない挙動のベースラインを作成すべきである。これを可能とするには以下が必要となる。

**必須メトリクスに関する推奨事項（SM-DR23）**: 外部クライアントやマイクロサービスの呼び出しのためにサービスメッシュを用いて収集するメトリクスの設定には，最低限以下の項目を加えるべきである。（a）一定期間内のクライアントやサービスのリクエスト数，（b）ステータスコード別に取得したクライアントやサービスの失敗したリクエスト数，（c）サービスごとの平均レイテンシ，および完全なリクエストライフサイクルごとの平均合計レイテンシ（理想的にはヒストグラムかつエラーコード別に）。

トレーシングは，一連の因果関係のある分散イベントを表したものであり，分散システムを通したエンドツーエンドのリクエストフローを符号化したものである[\[11\]][11]。
トレーシングは、リクエストが通過する経路（経由した様々なマイクロサービス）およびリクエストの構造（リクエストの分岐制御やその性質（同期/非同期））の両方を可視化できる。
マイクロサービス型アプリケーションおよびサービスメッシュの文脈において、トレーシング機能は様々なアプリケーションサービスやそれに関連するサービスプロキシの組み合わせで実現されるため，分散トレーシングと呼はれる。
定常状態における問題のデバッグや，キャパシティプランニングのためのデータ提供で使用されるため、分散トレーシングはアプリケーションシステムの可用性に対し，直接影響を及ぼしている。

**分散トレーシングの実装に関する推奨事項（SM-DR24）**: 分散トレーシングを実現するサービスプロキシの設定時，アプリケーションサービスは受信した通信パケットのヘッダを転送するよう実装されるよう注意すべきである。

上記の推奨事項を実行するには、サービスメッシュアーキテクチャ[\[14\]][14]配下で提供される他のインフラ機能とは異なり，最小限のアプリケーションサービスの改修が必要であることに注意することが重要である。

### 4.6 ネットワーク回復力技術の設定

**ネットワーク回復力機能のデータ保持に関する推奨事項（SM-DR25）**: リトライ、タイムアウト、サーキットプレイキングの設定やカナリアデプロイメントに関わるデータ(一般的にはコントロールプレーンの全構成データ)は、Key/Valueストアのような堅牢なデータストアに格納されるべきである。

**サービスインスタンスのヘルスチェックの実装に関する推奨（SM-DR26）**: サービスインスタンスのヘルスチェック機能は、負荷分散に使用される情報の整合性を維持するために、サービスディスカバリ機能と緊密に統合されるべきである。

このヘルスデータは中央ヘルスチェックサービス（もしくは中央サービスディスカバリサービス）に報告するこ��もできるが，ローカルでのみ使用することもできる（例えば，サービスプロキシが維持するオープンコネクションにヘルスチェックを行い，ローカルな負荷分散先を決定し，不健康と判断したホストを回避できる）。

### 4.7 オリジン間リソース共有（CORS）の設定

**CORS（オリジン間リソース共有）に関する推奨事項（SM-DR26）**: エッジサービス（すなわち，マイクロサービスのエントリーポイント）は，WebUIのクライアント側サービスのような外部サービスと通信するために，しばしばCORS設定が必要になる[\[15\]][15]。
エッジサービスのCORSポリシーはマイクロサービスアプリケーションのサービスコードを通して処理するのではなく，サービスメッシュの機能（例，IstioのVirtualServiceリソースのCorsPolicy設定）を使用して設定する必要がある。

### 4.8 管理操作の許可設定

**管理操作のためのアクセス制御に関する推奨事項（SM-DR28）**: サービスメッシュ内のすべての管理操作(例えば、ポリシー仕様、サービス回復力やカナリーデプロイメント、リトライなど設定パラメータ、)に対する、ネームスペース内の全サービスや特定サービスなどのレベルで指定可能な、きめ細かいアクセス制御権限があるべきである。一般に、この機能を行使するためのインタフェースは、サービスメッシュ自体の一部である必要はなく、アプリケーションサービスクラスタを構成するインストールソフトウェアやオーケストレーションソフトウェアの一部であってもよい。

## 5 概略および結論

クラウドや大規模なエンタープライズ環境におけるマイクロサービス型アプリケーションの採用が増加していることから、包括的で一貫性があり、調整された一連のサービス支援機能を提供するインフラストラクチャが求められている。
サービスメッシュはそのようなインフラストラクチャの1つである。現状のサービスメッシュのコンポーネントデプロイは、アプリケーションコードに変更を加えず全てのサービス支援機能を提供できるプロキシベースのアプローチである。

プロキシベースのサービスメッシュは、セキュアなサービスディスカバリ、認証と認可のポリシーの定義と実施、ネットワーク回復力機能、および性能とセキュリティの監視機能を提供するサービス支援コンポーネントを構築および統合するよう設計されている。本文書の主要な貢献は、上記の領域に列挙された各サービスメッシュコンポーネントの詳細なデプロイガイドを提供することである。

## 参考

1. Chandramouli R （2019） Security Strategies for Microservices-based Application Systems. （National Institute of Standards and Technology, Gaithersburg, MD）, NIST Special Publication （SP） 800-204. https://doi.org/10.6028/NIST.SP.800-204
2. Indrasiri K （2017） Service Mesh for Microservices. Available at https://medium.com/microservices-in-practice/service-mesh-for-microservices-2953109a3c9a
3. Lerner A, Thomas A （2018） Innovation Insight for Service Mesh. Available at https://www.gartner.com/en/documents/3894156/innovation-insight-for-service-mesh
4. Morgan W （2017） What’s a service mesh? And why do I need one? Available at https://buoyant.io/2017/04/25/whats-a-service-mesh-and-why-do-i-need-one/#_ga=2.39876518.581209135.1575405453-1411561046.1575405453
5. Mishra A （2017） Smart Networking with Consul and Service Meshes. Available at https://www.hashicorp.com/blog/smart-networking-with-consul-and-service-meshes/
6. Schiesser M （2019） Comparing Service Meshes: Linkerd vs. Istio. Available at https://medium.com/glasnostic/comparing-service-meshes-linkerd-vs-istio-c7e0132578a8
7. Bryant D （2018） The Importance of Control Planes with Service Meshes and Front Proxies. Available at https://blog.getambassador.io/the-importance-of-control-planes-with-service-meshes-and-front-proxies-665f90c80b3d
8. Kirschner E （2017） Proxy Based Service Mesh. Available at https://medium.com/@entzik/proxy-based-service-mesh-96cd4b74c198
9. Tiwari A （2017） A sidecar for your service mesh. Available at https://www.abhishek-tiwari.com/a-sidecar-for-your-service-mesh/
10. Sridharan C （2018） The Three Pillars of Observability. Distributed Systems Observability（O’Reilly Media, Inc., Sebastopol, CA）, Chapter 4. Available at https://www.oreilly.com/library/view/distributed-systems-observability/9781492033431/ch04.html
11. Jaeger Project （2020） Jaeger: open source, end-to-end distributed tracing. Available at https://www.jaegertracing.io/
12. Zipkin Project （2020） Zipkin. Available at https://zipkin.io/
13. Leong A （2019） A guide to distributed tracing with Linkerd. Available at https://linkerd.io/2019/10/07/a-guide-to-distributed-tracing-with-linkerd/
14. Stafford GA （2019） Kubernetes-based Microservice Observability with Istio Service Mesh: Part 1 （ITNEXT）. Available at https://itnext.io/kubernetes-based-microservice-observability-with-istio-service-mesh-part-1-bed3dd0fac0b
15. Stafford GA （2019） Istio Observability with Go, gRPC, and Protocol Buffers-based Microservices （ITNEXT）. Available at https://itnext.io/istio-observability-with-go-grpc-and-protocol-buffers-based-microservices-d09e34c1255a

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