# 非機能要件ヒアリング

## 可用性
### 稼働率

| 質問                                                   | 質問の意図                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | ご回答 |
| ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------ |
| システムごとの重要度を設定していますか？               | システムが提供している機能ごとに重要度が違う。重要度ごとに運用レベルを定義する。                                                                                                                                                                                                                                                                                                                                                                                                     |        |
| システムごとの運用時間を定義していますか？             | オンライン/バッチを含みシステムが稼働している時間を定義する。メンテナンスは運用時間を避けて実施するよう設計する。「とりあえず24時間」のようなシステムは多いが、AWS の場合はユーザーが避けられないメンテナンスがあるので「とりあえず24時間」は金額がかさむことを丁寧に説明すること。                                                                                                                                                                                                  |        |
| システムごと、重要度に応じた稼働率を定義していますか？ | オンプレミスでの稼働率は考えないことにする。AWSのベストプラクティスに合わせた現実的な稼働率を定義し直す。<br />EC2 コントロールプレーンの可用性設計目標 99.950%、EC2 Single-AZ データプレーンで 99.950%、EC2 Multi-AZ データプレーンで 99.990%、<br />RDS コントロールプレーンで99.950%、RDS Single-AZ データプレーンで99.950%、RDS Multi-AZ データプレーンで99.990%、<br />もし本当に稼働率を 99.99% 以上にしたいなら Multi-AZ 必須なうえに障害から自動で復旧させる仕組みを考える。 |        |
| 定期的な計画停止を定義していますか？                   | ・AWSが行うメンテナンスがあり、場合によっては避けられない<br />・スケールアップ/ダウンは原則としてインスタンス停止になる<br />・重要なパッチの適用に再起動が伴う<br />このようなケースでも稼働率を下げないように月次または年次の計画停止を盛り込んでおく。計画停止はメンテナンスの予定が無ければスキップしてよい。                                                                                                                                                                   |        |

### バックアップ / リストア

| 質問                                                                                                                                              | 質問の意図                                                                                                                                                                                                                                                                                                             | ご回答 |
| ------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| RPO（目標復旧時点）は定義されていますか？                                                                                                         | スナップショット取得間隔に関係する。<br />10分前に戻りたいなら10分おきにスナップショットを取得するが、コストとのバランスをとること。<br />RDBMS on EC2 の場合は、製品ネイティブのバックアップ機能で RPO を準拠できるようセッティングを行う。                                                                           |        |
| RTO（目標復旧時間）は定義されていますか？                                                                                                         | インフラをリストアする時間だけではなく、アプリケーションをリストアする時間も考慮する。<br />例えば、スナップショットからリストア後に最新バージョンのデプロイが必要ならその所要時間や手順自動化も考えておく。<br />障害原因特定に要する時間は制御が難しいと思われるので、復旧作業の時間に関する目標を確認してはどうか？ |        |
| RLO（目標復旧レベル）は定義されていますか？                                                                                                       | どこまでリストアできれば復旧と言えるかを定義する。<br />例えば、主要な機能のリストアが数時間、補助機能は翌日という考えもあり。<br />現実的なレベルにしておくこと。                                                                                                                                                     |        |
| システムの重要度に合わせて RPO、RTO、RLOは定義されていますか？                                                                                    | RPO、RTO、RLOの実現はコストに直結するため、システム重要度に合わせて定義する。                                                                                                                                                                                                                                          |        |
| バックアップに対する要件をご教示ください。<br />・インシデントに備えたバックアップ<br />・ファイル保管要件に対応するバックアップ                  | 障害に備えるバックアップもあれば、ファイルの長期保管に備えるバックアップもある。<br />どのようなバックアップが必要なのかを明らかにしておく。                                                                                                                                                                           |        |
| 保存期間 / 世代は定義されていますか？<br />ただし、コンプライアンス対応のためのアーカイブとは混同しないように留意                                 | インフラの障害によるボリュームの破損はすぐに気がつけるが、アプリケーションのバグや不備によるデータの不整合などは発見しにくいと思われる。                                                                                                                                                                               |        |
| データをリストアするときの粒度はどのレベルになりますでしょうか？<br />・ボリューム<br />・ファイル<br />・データベース (スキーマ、テーブル、全体) | リストアの粒度に応じてバックアップ手段を検討する。<br />データベースは製品ネイティブな機能でエクスポートしてもらう。そのタイミングとAWSでのバックアップタイミングは足並みを揃える。                                                                                                                                    |        |

### 災害対策

| 質問                                                                 | 質問の意図                                                                                                                                                                                                                                                                                                                                                                                             | ご回答 |
| -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------ |
| DR/災害対策を講じる必要はございますか？                              | Multi-AZ・Multi Regionなど、構成を決めるためのインプット。<br />リージョンレベルの冗長化だけでなく、オペレーションの体制も検討する必要がある。                                                                                                                                                                                                                                                         |        |
| DRが発動する「災害」の定義を教えてください。                         | 何をもってしてDR発動とするかを聞く。対策のレベル感が決まる。 <br /> - 大規模な自然災害による社会インフラ (電気やネットワークなど) の停止<br /> - 大規模な自然災害による自社建屋の崩壊 (ハイブリット構成、Direct Connect の前提など)<br /> - 人的エラーによるデータ損失、サービス停止<br /> - AWS 起因による特定リソースの停止<br /> - 悪意のある第三者による攻撃を受けたサービス停止やサイト改ざんなど |        |
| DR復旧時間目標を教えてください。                                     | 災害の定義によって復旧時間は異なる。<br />関東近郊の大規模災害 (東京リージョン全滅) なのに数時間で復旧などは相当な費用がかかる。                                                                                                                                                                                                                                                                       |        |
| DR復旧レベルを教えてください。                                       | DR発動時、どのサービスがどのレベルまで復旧できていればよいか？<br />どのレベルまで復旧させるかはコストに関わってくる。                                                                                                                                                                                                                                                                                 |        |
| DR事象発生後、切り替え実施を判断するまでの猶予時間をご教示ください。 | Active-Active な DR 構成なら猶予時間を短くできる。 <br />DR発生でも通常ロケーションで復旧を試みるほうが復旧が速いケースはある。                                                                                                                                                                                                                                                                        |        |

### コンプライアンス

| 質問                                                                                                                      | 質問の意図                                                                                                                                                                                                                                                                                                                                                                                                                                                          | ご回答 |
| ------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| 貴社業務における組織要件、法的要件、コンプライアンス要件を教えてください。                                                | 業界固有のセキュリティ要件、グループ会社で遵守すべきセキュリティ要件などがあるので事前に入手しておく。                                                                                                                                                                                                                                                                                                                                                              |        |
| 定期的なセキュリティチェックを行いますか？そのチェックはどのような内容が必要でしょうか？                                  | Security Hub、Conformance Packs、Trusted Advisor などで定期的なチェックを行う。<br />要件によっては 3rd Party Tool を検討する。                                                                                                                                                                                                                                                                                                                                     |        |
| 外部ベンダーによる定期的な脆弱性チェックは必要でしょうか？                                                                | 業界標準によっては定期的な脆弱性チェックが必要になる。                                                                                                                                                                                                                                                                                                                                                                                                              |        |
| AWSマネジメントコンソールへのログインに際し、MFAデバイス(仮想/物理)を使用できますか？<br />多要素認証を推奨しております。 | セキュリティ強化のためMFAを義務化する。                                                                                                                                                                                                                                                                                                                                                                                                                             |        |
| AWSマネジメントコンソールへのログインに際し、パスワードポリシーの設定は必要ですか？                                       | IAM パスワードポリシーを強制する。                                                                                                                                                                                                                                                                                                                                                                                                                                  |        |
| 認められていないAWSリソースへの操作に対して通知を行いますか？                                                             | CloudTrail のログを監視。<br />例: Amazon S3 バケットのアクティビティ<br />例: セキュリティグループの設定の変更<br />例: ネットワークアクセスコントロールリスト (ACL) の変更<br />例: ネットワークゲートウェイの変更<br />例: Amazon Virtual Private Cloud (VPC) の変更<br />例: Amazon EC2 インスタンスの変更<br />例: EC2 ラージインスタンスの変更<br />例: CloudTrail の変更<br />例: コンソールサインインの失敗<br />例: 認証エラー<br />例: IAM ポリシーの変更 |        |
| 保管するデータの機密分類はありますか？ある場合、どのようなデータをどのように保護するか教えてください。                    | 暗号化や IAM/S3ポリシーに関わる。<br />機密分類が高いデータはバケットを分ける等の対策を講じる。                                                                                                                                                                                                                                                                                                                                                                     |        |
| 転送中のデータの機密分類はありますか？<br />ある場合、どのようなデータをどのように保護するか教えてください。              | TLS通信を用いる範囲はどこまでなのか、AWS外はTLS必須にする、VPC外はTLS必須にする、VPC内は任意など。                                                                                                                                                                                                                                                                                                                                                                  |        |
| 暗号化キーの保管について要件はありますか？                                                                                | ユーザーで作成した暗号化キーを使うか、AWSデフォルト暗号化でよいか。<br />PCI DSS は KMS 必須。                                                                                                                                                                                                                                                                                                                                                                      |        |
| 業界・組織で起こりがちなセキュリティインシデントは？<br />また、過去に発生したセキュリティインシデントは？                | 脅威モデルを定義する場合のインプット、再発防止。                                                                                                                                                                                                                                                                                                                                                                                                                    |        |

### パッチ管理

| 質問                                                               | 質問の意図                                                                                                                                           | ご回答 |
| ------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| 組織のセキュリティポリシーで、パッチ管理に関する規定はありますか？ | 設計のインプット（前提条件として明記）とするため。                                                                                                   |        |
| 既存のパッチ管理システムは存在しますか？                           | あれば、設計ではワークロードを管理対象にするか検討する。                                                                                             |        |
| 既存のパッチ管理システムでワークロードを管理しますか？             | 既存のパッチ管理をそのまま使う場合は使う前提で設計、使わない場合は Patch Manager を検討。                                                            |        |
| パッチ管理の対象となるプラットフォームはご教示ください？           | Windows、主要Linuxは問題なさそうとして、マイナーOSをどうするか検討。                                                                                 |        |
| 重大な脆弱性を緊急パッチ適用の対象とする運用を行いますか？         | 行う場合、検知の仕組みと作業するための体制を検討する。                                                                                               |        |
| 緊急パッチと判断する条件に指定がありますか？                       | 基本的には OS ベンダーが規定している「緊急」を採用。<br />MWも同じ考えでいい。CVSS Scoreを採用してもいい。                                           |        |
| パッチ適用前のテストを実施していますか？                           | していない場合には検証環境の用意が現実的か確認し、可能なら準備することを推奨。<br />ライセンスなどの問題で検証環境の構築が現実的ではない場合がある。 |        |
| パッチ適用のためのメンテナンスウィンドウを確保できますか？         | パッチを確実かつ安全に適用するならメンテナンスウィンドウを設ける。                                                                                   |        |

### 攻撃防御

| 質問                                                                                                           | 質問の意図                                                                                                         | ご回答 |
| -------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | ------ |
| アンチウィルス/マルウェアの導入は必要でしょうか？<br />また、既存でお使いの製品を継続使用をしますか？          | EC2 に対応した製品を選定する。コンテナならAquaなど。                                                               |        |
| IDS/IPS の導入は必要でしょうか？                                                                               | インターネットと接する箇所にAWS WAFを導入する。<br />VPC内部でIDS/IPSが必要な場合はEC2上で稼働する製品を選定する。 |        |
| 既存で IDS/IPS を導入されている場合はルールを提供ください。                                                    | 同等や類似機能を実装する際の材料とする。                                                                           |        |
| 接続元制限の要件をご教示ください。<br />・既存の IP アドレス許可/拒否リスト<br />・国による接続許可/拒否リスト | 同等や類似機能を実装する際の材料とする。                                                                           |        |

### 運用アカウント管理

| 質問                                           | 質問の意図                                                                                             | ご回答 |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------ | ------ |
| どのような体制でサービスの運用を実施しますか？ | 体制と作業分担を明らかにする。<br />また、作業を実施するために必要な権限をロールに割り当てる。         |        |
| 各ロールに割り当てられるメンバーは誰？         | ロールを割り当てるユーザを決定するため                                                                 |        |
| パスワードポリシーは定義されていますか？       | ISMS等で定義されている場合にはそれにのっとる                                                           |        |
| 職責に応じた権限移譲が定義されていますか？     | 役職ではなく職責で権限が定義されているか？<br />通常アクセスしない人間に強い権限が与えられていないか？ |        |

## 運用
### ジョブ管理

| 質問                                                                                 | 質問の意図                                                                                                                                                                                  | ご回答 |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| ジョブ管理ツールの導入は必要でしょうか？                                             | ジョブネット、業務カレンダーなどのジョブ管理ツール特有の機能が必要であれば有償製品を選定する。<br />cron なら CloudWatch Events、ステートでのジョブ管理ならば Step Functions が活用できる。 |        |
| ジョブ管理ツールに求める機能をご教示ください。                                       | 同上。                                                                                                                                                                                      |        |
| ジョブの実行に失敗した場合、通知は必要ですか？<br />また、どこに通知するべきですか？ | リトライにも失敗した場合には手動での実行や原因調査が必要なため、原則通知するべき。誰に通知するべきかは体制次第。                                                                            |        |

### ログ管理

| 質問                                                                                                                                                                                                                                                                                                                                                                                    | 質問の意図                                                                                                                        | ご回答 |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- | ------ |
| ログ管理が必要でしょうか？<br />管理対象のログをご教示ください。<br />・OS<br />・ミドルウェア（EC2上のログファイル、RDS、Elasticsearch Service 等）<br />・アプリケーション（EC2上のファイル、Lambda 等）<br />・ネットワーク（CloudFront、WAF、ELB、FlowLogs 等）<br />・AWS（CloudTrail、VPC Flow Logs、S3、RDS 等）<br />・セキュリティ（GuardDuty、Config Rules、Abuse Report 等） | 何をどこに保管するか。<br />AWSではどんなログが取得できて、どう活用するかの説明が必要。                                           |        |
| それぞれのログの利用目的をご教示ください。                                                                                                                                                                                                                                                                                                                                              | 監査目的、障害調査、アクセス分析などログごとの目的を知り、最適な保管場所を決定する。                                              |        |
| 保管期間は定義されていますでしょうか？                                                                                                                                                                                                                                                                                                                                                  | 保管要件を遵守する。また保管期間はAWS利用料金に影響してくる。                                                                     |        |
| ログ分析の必要はありますでしょうか？<br />どのログをどの粒度で分析するか定義されていますでしょうか？                                                                                                                                                                                                                                                                                    | Athena、CloudWatch Logs Insight、QuickSight、Amazon ES、3rd party tool などの選択肢がある。<br />セキュリティ系ログならSIEMなど。 |        |

### 監視

| 質問                                                                                                                                                                     | 質問の意図                                                                                                                                                                                              | ご回答 |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| リソース監視を行いますか？<br />その場合の指標はございますか？<br />CPU使用率/メモリ使用率/ディスク使用率/アクセス数/転送量など、AWSリソースに対する監視を示しています。 | CloudWatch & Agent で完結するか、しなければ 3rd party tool を検討。                                                                                                                                     |        |
| OS上のサービスまたはプロセス監視を行いますか？<br />その場合の指標はございますか？<br />AWSマネージドサービスやサーバーレスを使う場合は対象外です。                      | CloudWatch & Agent で完結するか、しなければ 3rd party tool を検討。                                                                                                                                     |        |
| AWSリソースの死活監視を行いますか？                                                                                                                                      | ping 監視は効果が薄い。PHDやCloudWatchで死活を監視する。                                                                                                                                                |        |
| フロントエンド監視を行いますか？<br />その場合の指標はございますか？                                                                                                     | 外形監視はサービスの稼働状態を知るために必要。                                                                                                                                                          |        |
| アプリケーション監視を行いますか？<br />その場合の指標はございますか？                                                                                                   | 外形監視で一気通貫に確認できるとよい。<br />アプリケーションは個別に監視しないとならないことが多いのでヒアリングしておく。                                                                              |        |
| ビジネスKPIを計測していますか？                                                                                                                                          | ビジネスKPIで必要な指標に対して、ITとしてどのメトリクスを提供すべきか。                                                                                                                                 |        |
| 通知するメトリクス、しきい値は定義されていますでしょうか？                                                                                                               | オオカミ少年にならないように通知は必要最低限に留める。<br />通知先は緊急と通常の2通り用意する。<br />サービスダウンなど人間による即時対処が必要なものを緊急にする。日中帯の対応でよいものは通常にする。 |        |
| 通知方法はどのような想定でしょうか？<br />メール、チャットツール、電話、チケットツール、統合管理ツールなど。                                                             | AWSから通知先への手段はアーキテクチャ設計に関わる。<br />メールやSlackは実装容易、電話やチケットツールへの連携は構築作業が必要。                                                                        |        |
| 通知すべきイベントと通知先は定義していますか？<br />（「障害」を定義していますか？）                                                                                     | イベントが対処すべきチームへ適切に届くようにする。<br />そもそも、何を障害と見なすか定義するべき。                                                                                                      |        |
| アラート通知に重要度は設定していますか？                                                                                                                                 | 緊急アラートと参考情報アラートを分離して、アラート自体がオオカミ少年にならないようにする。                                                                                                              |        |
| アラート重要度に応じて通知先を定義していますか？                                                                                                                         | 緊急は電話、参考情報はメールなど。                                                                                                                                                                      |        |

### アプリケーションデプロイ

| 質問                                                                         | 質問の意図                                                                                 | ご回答 |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ | ------ |
| アプリケーションのビルド方法について、クラウド最適化する予定はございますか？ | デプロイの方法・問題発生時のロールバック方法・テスト方法などの定義を行う必要があるか確認。 |        |
| アプリケーションのデプロイについて、クラウド最適化する予定はございますか？   | デプロイの方法・問題発生時のロールバック方法・テスト方法などの定義を行う必要があるか確認   |        |
| アプリケーションコードのバージョン管理ツールは使用していますか？             | AWSサービスとの統合を考慮。                                                                |        |
| アプリケーションのテストを自動化するツールを使用していますか？               | AWSサービスとの統合を考慮。                                                                |        |
| デプロイのロールバックを容易にしていますか？                                 | デプロイの方法・問題発生時のロールバック方法・テスト方法などの定義を行う必要があるか確認   |        |


### 運用維持管理

| 質問                                                                                                       | 質問の意図                                                                                                                                                                                                                  | ご回答 |
| ---------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| 基盤運用および運用管理を実施する体制を教えてください。（CCoE / SREチーム / インフラ・基盤担当チーム など） | 設計や実装のスコープについて「現実的な範囲」を検討するためのインプットとする。                                                                                                                                              |        |
| （上記のメンバーについて）AWSに関するスキルはどの程度お持ちでしょうか？                                    | 必要に応じてトレーニングの受講を提案する。                                                                                                                                                                                  |        |
| 課題解決や障害対応、定常作業などのタスクを管理する仕組みはありますか？                                         | ある場合にはそのツールを前提のプロセスを設計する。<br />ない場合にはBacklogやJIRAなどのタスク管理サービスの利用を推奨。<br />成果を可視化することは運用の価値を評価するためにも必要。                                       |        |
| 定期的な運用レビューを実施する会議体はありますか？                                                         | ない場合には、設置の推奨と形骸化しないための運営方法を提案。                                                                                                                                                                |        |
| 業務上有益なナレッジ・手順・障害報告を共有する手段がありますか？                                           | ある場合にはそのツールを前提のプロセスを設計する。<br />ない場合、GitHub / Confluenceなどのサービスの利用を推奨。<br />障害やインシデントに関しては、レポートのフォーマットやレビューなどを定義して継続的改善を可能にする。 |        |
| システムメンテナンスに必要な作業は手順書化されていますでしょうか？                                         | AWSで新たに発生する運用手順を手順書化する。                                                                                                                                                                                 |        |
| 手順書を自動実行するツールを使用していますか？                                                             | ツールをAWSでも継続利用可能か？ RDSやLambdaなどに対応しているか？                                                                                                                                                           |        |
| メンテナンス作業を行うにあたりプロセスフローは定義されていますか？                                         | プロセスフローがクラウド運用にマッチしているものか？マッチさせる必要はあるか？                                                                                                                                              |        |
| システムイベントへの対応を自動化していますか？                                                             | イベントの復旧アクションを自動化しているか、する要件はあるか？                                                                                                                                                              |        |

## キャパシティ
### 性能目標値

| 質問                                                                         | 質問の意図                                                                                   | ご回答 |
| ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- | ------ |
| オンライン処理、バッチ処理に求められている性能目標値があればご教示ください。 | アーキテクチャやインスタンスタイプ、ボリュームタイプ選定に影響するようであれば把握しておく。 |        |
| 保管するデータ量、データベースのデータ量をデータ種別ごとにご教示ください。   | アーキテクチャやインスタンスタイプ、ボリュームタイプ選定に影響するようであれば把握しておく。 |        |
| 必要とする IOPS をアプリケーションごと、データベースごとにご教示ください。   | アーキテクチャやインスタンスタイプ、ボリュームタイプ選定に影響するようであれば把握しておく。 |        |
| 外部システム連携がある場合、必要なネットワーク帯域をご教示ください。         | 常時一定の帯域が必要な場合は帯域保証の Direct Connect を検討。                               |        |

### 拡張性

| 質問                                                                                                                                                                            | 質問の意図                                                                                                                         | ご回答 |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ------ |
| スケールアップ/ダウンを行う指標はございますか？                                                                                                                                 | リサイズ方式に影響する。基本的にリサイズにはサービス停止が伴う。                                                                   |        |
| スケールイン/アウトを行えるようなアプリケーションを採用していますか？                                                                                                           | AutoScaling に対応しているかはアーキテクチャに影響する。また、マネージドサービスのスケールに対応できるアプリケーションが望ましい。 |        |
| アプリケーションごとの負荷特性をご教示ください。<br />・季節によって負荷が変わる<br />・テレビや SNS で宣伝すると負荷が急上昇する<br />・月次処理が走ると負荷が上がる<br />など | スケーリング方式に影響する。いつどのようなタイミングでスケールすべきかを予め把握する。                                             |        |
| 性能テストの要否をご教示ください。                                                                                                                                              | テスト方式に影響する。                                                                                                             |        |

## 環境
### ライセンス管理

| 質問                                                                                                                                 | 質問の意図                                                                                                     | ご回答 |
| ------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------- | ------ |
| AWSへ持ち込むライセンスをご教示ください。<br />また、AWSで稼働させることについてライセンス販売元へのサポート有無確認をお願いします。 | 管理対象を特定したい。<br />「そもそも持ち込めるのか」を確認。クラウド上ではライセンス体系が変わるものもある。 |        |
| 移行期間中の並行稼動が許容されているかライセンス販売元へご確認ください。                                                             | 移行による並行稼動は期間限定で許容してくれるパターンもあり。                                                   |        |

### AWSアカウント

| 質問                                                                                        | 質問の意図                                                                                   | ご回答 |
| ------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- | ------ |
| 請求分離のためAWSアカウントを複数作成しますか？                                             | AWS利用料金を詳細まで正確に分離するには、請求単位でAWSアカウントを作成する。                 |        |
| システム環境(開発/ステージング/本番 など) ごとにAWSアカウントを複数作成しますか？           | セキュリティ面での統制を強化するために環境ごとにAWSアカウントを作成する。                    |        |
| ログ収集アカウントやセキュリティ管理、ネットワーク管理だけを行うAWSアカウントが必要ですか？ | 複数アカウントを使用する場合、特定目的のアカウントを作成して集中管理をしたほうが効率が良い。 |        |

### ネットワークトポロジ

| 質問                                                                                                      | 質問の意図                                                                             | ご回答 |
| --------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- | ------ |
| Direct Connect を使用してAWSと接続するAWSアカウントはありますか？                                         | 回線冗長化、IPアドレス管理、複数アカウントの場合の接続などアーキテクチャに関する考慮。 |        |
| 貴社拠点 ～ Direct Connect ～ AWSの想定接続図をご提供ください。                                           | Direct Connect Gateway、Transit Gateway など最適なトポロジを考慮。                     |        |
| VPN を使用してAWSと接続するAWSアカウントはありますか？                                                    | 回線冗長化、IPアドレス管理、複数アカウントの場合の接続などアーキテクチャに関する考慮。 |        |
| 貴社拠点 ～ VPN～ AWSの想定接続図をご提供ください。                                                       | Transit Gateway など最適なトポロジを考慮。                                             |        |
| 貴社システムから接続する他システムで IP アドレスや FQDN で接続元制限をかけている接続先はありますか？      | IP アドレスを固定するために NAT Gateway やプロキシサーバーの設定を検討する。           |        |
| 他システムから貴社システムへ接続する場合、IP アドレスや FQDN で接続元制限をかけている接続先はありますか？ | IP アドレスを固定するために NAT Gateway やプロキシサーバーの設定を検討する。           |        |
