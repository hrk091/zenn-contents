---
title: "CloudNative Days Tokyo 2022 まとめ"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CloudNativeDays", "Kubernetes"]
published: true
---

2022年11月21日-22日の2日間で、CloudNative Days Tokyo 2022が開催されました。

面白い発表がたくさんあったので、記憶が薄れないうちにざっくりとまとめてみます。
個人的に印象に残ったところを切り取っただけの雑なまとめなので、気になった方はぜひ発表を見てみてください。以下のページからアーカイブ視聴が可能になっています。

https://event.cloudnativedays.jp/cndt2022/talks

# 1日目

## Kubernetesのnamespaceを越えて: namespace跨ぎのsnapshotからのvolumeの作成の今と未来

https://event.cloudnativedays.jp/cndt2022/talks/1521

KubernetesのVolumeSnapshotリソースが他のnamespaceからでは参照できずデータ共有ができない、という問題を解決するCrossNamespaceVolumeDataSource(KEP-3294)と、この問題に対してコントリビュートする取り組みの紹介でした。
概要の説明と過去の失敗案の課題についてもしっかり言及されていて、学びが大きいです。
長きに渡る戦いを繰り広げた結果、AnyVolumeDataSourceによる拡張とReferenceGrantによるアクセス許可で解決する方向に進んでいるとのことです。
GatewayAPIリソースであるReferenceGrantが転用できるというところがなるほどなあという感じでした。


## GKE の Spot VM / Preemptible VM 利用者を救う！インフラコストを削減しながら可用性を維持できるソフトウェア『Panope』

https://event.cloudnativedays.jp/cndt2022/talks/1567

GKEのワーカーノードとしてPreemptible VM・Spot VMを使用した際に、可用性を高めるソリューションであるPanopeというプロダクトの紹介でした。
Spot VMを使っている場合に、インスタンスの停止を予測して事前にdrainしてノードをシャットダウンしておくという発想が素晴らしいなと思いました。
他にも、GCE側の在庫枯渇時は通常のNode Poolを一時的に作成し、在庫枯渇が終了してから再度Spot VMに移行させるなど、実践的なユースケースがよく考えられています。
円安でインフラのコストが大きな課題となっている現在、可用性高くSpot VMを運用できると嬉しいユーザは多いハズで、このプロダクトは引き合いが強そうです。


## Self-Hosted Runners & Actions Runner Controller (ARC)を運用すること

https://event.cloudnativedays.jp/cndt2022/talks/1553

Kubernetesカスタムコントローラであるactions-runner-controllerを用いて、Github ActionsのSelf-Hosted Runnerを構築・運用する話です。
actions-runner-controllerについて詳細に説明されていて、勉強になります。 KubernetesのPod・Replicaset・Deploymentと同じようなアーキテクチャで構成されているとのことです。
個人的には同じくCIプラットフォームであるTektonとの違いが気になりましたが、ワークフロー管理はGithub側で行われていてワーカーリソースの提供に徹しているので、ワークロードを扱う機能に特化して提供されているようです。
なお、actions-runner-controllerについては、[ChatWorkさんの講演](https://event.cloudnativedays.jp/cndt2022/talks/1532)でも言及されていました。


## 明日から始められるKyvernoを用いたポリシー制御

https://event.cloudnativedays.jp/cndt2022/talks/1568

Kyvernoについて大変分かりやすく説明されていました。
ちょうど知りたいと思っていたところなので、ありがたいセッションでした。
Kyvernoについて日本語文献でざっと概要を理解したい方は、この講演を聴くと良さそうです。


## Rook/Cephストレージシステムを開発しながらupstream OSSに成果を還元してきた取り組み

https://event.cloudnativedays.jp/cndt2022/talks/1525

Rook/Cephの開発・検証の話に加えて、OSSのメンテナとしての活動、コミュニティとの付き合い方について紹介されていました。
一線でご活躍されている経験から、オープンソースに貢献しコミュニティとうまく連携するために大事なことを展開していただけるのは大変ありがたいです。
OSPO（Open Source Program Office）についての言及がありますが、本日開催のOpen Source Summit Japan 2022でOSPOのマイクロカンファレンスが開催されていますので、詳しく知りたい方はそちらも見ると良さそうです。


## 分散合意を用いたクラウドネイティブトランザクション Paxos Commit

https://event.cloudnativedays.jp/cndt2022/talks/1573

2-Phase Commitでも、単一障害点となるコーディネーターを冗長化すればトランザクションの消失に伴う原子性の崩壊を避けることができ、そのためにPaxos Commitが活用できる、という話です。
マイクロサービスのような分散システムにおけるトランザクションにはSagaパターンを使うのが良いとされていますが、Sagaパターンには様々な課題があり、完全なConsistencyを実現しようとすると 「補償トランザクションによるビジネスルールの汚染」「トランザクションの永続化」「セマンティックロックの導入」など課題が多くめんどくさいです。
私もSagaパターンの実装は極力やりたくないです。
この提案では、整合性(C)と分断耐性(P)を優先しても、Paxos/Raftなどの分散合意アルゴリズムを使えば可用性(A)を100%に近づけられる、という理論に基づいています。
とてもわかりやすく、なるほどなあと思える良い発表でした。


## kubefork - 自分専用のクラスタを所有しているような開発体験

https://event.cloudnativedays.jp/cndt2022/talks/1590

Istio + Telepresenceを用いて開発中のコンテナだけcopy on writeできるようにすることで、共用のクラスタにも関わらず自分の開発中のコンテナだけ入れ替えて検証ができる、kubeforkというKubernetesカスタムオペレータを開発しました、という話です。
過去にも類似の発表をなさっているようですが、今回はOSSとして公開したと報告がありました。
ただし利用するには、IstioでroutingできるようにHeaderを挿入する形で通信するように各コンテナを作り込む必要があるとのことで、kubeforkに対応するための追加開発が必要なようです。
開発環境を過度に増やすことなく置き換えたい箇所だけトラフィックを曲げて検証できるようになるのは、誰もが一度は夢見た機能ではないかと思うので、それを実現しているのは素晴らしいと思いました。 今後のさらなる改良に期待したいです。


## SLO策定のIaC化によるSREの加速

https://event.cloudnativedays.jp/cndt2022/talks/1580

SLOについての考え方の整理と、SLOのIaC、すなわちSLOの宣言文書からSLOの計装や可視化などを自動生成するためのツールおよび標準化、の動向についての紹介でした。
googleが主導しているSLO Generatorという実装とOpenSLOというフォーマットが積極的な動きを見せていて、フォーマットはOpenSLOが標準になりそうな動きらしいです。
SLOについてYAMLで宣言すると、PrometheusやDatadogが設定されてSLOが計測可能になる、という便利な世界が実現しつつあるようなので、引き続きウォッチしていきたいです。


# 2日目

2日目には[私の講演](https://event.cloudnativedays.jp/cndt2022/talks/1526)もありますが、ここでは特に紹介しません。講演資料のリンクを貼っておきますので、ご興味があれば見ていただけると嬉しいです。

https://speakerdeck.com/hrk091/cd-nosaaspurodakutohua-henotiao-zhan


## 同志諸君よ、ゼロトラストを撃て

https://event.cloudnativedays.jp/cndt2022/talks/1582

各クラウドプロバイダーのセキュリティソリューションを駆使し社内基盤もMicrosoftのEDRなどで構成し、最小権限の原則・職掌分離をしっかりやっていったら、ビジネス成長に伴う組織の拡大・多様化についていくのが大変だったという内容でした。
組織の成長への追従のためにガバナンスに関しても継続的にアップデートしないといけない（そしてそれが難しい）という話、どの分野でも共通だなと思いました。
こちらの発表ではゼロトラストをスケールさせる取り組みとして、ABACによるポリシーベースの動的なアクセス制御に切り替えるという話と、内部統制ポリシーを形骸化させないためにOSCALという評価フレームワークを導入するという話の2点が紹介されていました。


## SLO策定までの道とChaosEngineeringを使った最適解の見つけ方

https://event.cloudnativedays.jp/cndt2022/talks/1537

効果的なSLI/SLOを策定するための試行錯誤と、Chaos Meshを使ってSLO違反状態をシミュレーションして妥当性を検証してみた、という内容でした。
Chaos Engineeringの応用として、SLOが満たされることの検証ではなく、SLOが満たされていない状態のサービスを体感して納得感のある値を探索する、という用途に使う発想が面白いなと思いました。
Chaos Meshは良さそうなので、テストスイートに入れて使ってみたいです。


## Kubernetes Admission Webhook Deep Dive

https://event.cloudnativedays.jp/cndt2022/talks/1579

[つくって学ぶKubebuilder](https://zoetrope.github.io/kubebuilder-training/)のzoetroさんから、Admission Webhookに関するDeep Diveセッションです。
実装方法についての概要から、並行実行時の競合への対策、セキュリティ対策、マルチテナント対応、などライトユースだと問題にならないけど深堀りし始めると顕在化する問題に言及されており、大変参考になりました。カスタムオペレータ実装の参考にもなるので、早速活用したいと思います。
脱Admission Webhookとして紹介されていたCEL for Admission Controlはv1.26から任意のリソースに対してValidationできるようになるそうです。早く検証したいです。


## リアルタイム分析サービスをクラウドネイティブDB (TiDB) で作るとこうなる

https://event.cloudnativedays.jp/cndt2022/talks/1541

TiDBのユースケース・アーキテクチャについての紹介でした。
お恥ずかしながら、TiDB含めHTAP(Hybrid Transactional and Analytical Processing) という領域について知見がなかったので、勉強になりました。
SQL互換であり、TiKVからTiFlashへのコピーがalterで簡単にできて、どちらを使うかはクエリオプティマイザで自動判定される、というデモを見て、新時代を感じました。
登壇者の意に反しそうで恐縮ですがまずはさくっとマネージドで素振りしたく、AlloyDBを触ってみたいと思いました。
講演中で紹介されていた [https://ossinsight.io](https://ossinsight.io) も素晴らしいツールでした。


## ScalarDB: Cloud-native Transaction Manager

https://event.cloudnativedays.jp/cndt2022/talks/1550

クラウドネイティブな分散トランザクションマネージャである、ScalarDBの紹介です。 1日目に講演されていた、2-phase-commitにおけるコーディネーターを合意アルゴリズムで冗長化する機構を具備しており、RDBだけでなくNoSQLも対象としたACIDトランザクションを実現しています。
MySQLとDynamoDBのように、RDB-NoSQLでACIDを担保できるのがなかなか画期的です。
その分パフォーマンスは落ちるのですが、パフォーマンスが重要ではなくトランザクション特性がミッションクリティカルなアプリケーションにおいて、重宝しそうです。


## KeycloakでFAPIに対応した高セキュリティなAPIを公開する

https://event.cloudnativedays.jp/cndt2022/talks/1548

FAPI(Financial-grade API)と、OAuth/OIDC/SAML/ソーシャルログインなどの機能を揃えたIAMのOSSであるKeyclockの紹介でした。
本セッションでアピールしたいこととは逸れるかもですが、OAuth Authorization Code Grantを例として、未対策な場合に可能な攻撃とFAPIに準拠することでその攻撃をどう防御できるかについて、8つの具体事例を用いて丁寧に説明されているのが素晴らしかったです。
わかりやすく、大変ありがたいまとめでした。


# まとめ

CloudNativeDays Tokyo 2022で面白かったセッションを、主観たっぷりにまとめてみました。
他にも面白いセッションがたくさんありましたので、参加した方もできなかった方も、ぜひセッション一覧を見て、気になる動画のアーカイブを見てみてください！
