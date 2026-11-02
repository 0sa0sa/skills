# 進化的アーキテクチャ・適応度関数

フィットネスファンクション、ADR、段階的変更、Strangler Figなど進化的アーキテクチャの実践を見る。

## チェックリスト

### 1. 保護すべきアーキテクチャ特性ごとにfitness functionを定義し、Code Quality/Resiliency/Observability/Performance/Compliance/Security/Operabilityなど複数カテゴリに分けてビルド/デプロイメントパイプラインに組み込み、継続的に検証しているか
- **なぜ重要か**: fitness functionのない原則は『読まれないwiki』で終わり形骸化を防げず、機能テストのみでは品質特性全体の劣化を検知できないため
- **出典**: Building Evolutionary Architectures 2nd Ed 第2章全体, 第4章 p.78; Mastering API Architecture 第8章 p.198

### 2. fitness functionをscope(atomic/holistic)・cadence(triggered/continual/temporal)・result(static/dynamic)・invocation(automated/manual)・proactivity(intentional/emergent)の軸で分類し、運用負荷と即応性のトレードオフを意識して選んでいるか
- **なぜ重要か**: 各軸で運用負荷・即応性が異なり、分類せず導入すると過剰または不足したガバナンスになるため
- **出典**: Building Evolutionary Architectures 2nd Ed 第2章 p.18-26

### 3. fitness functionを定義する前に、変更が影響する『次元』(技術・データ・セキュリティ・運用等)を明示的に洗い出す3ステップの手順を踏んでいるか
- **なぜ重要か**: 次元の洗い出しが漏れると保護すべき特性そのものが欠落し、気づかれないまま劣化が進むため
- **出典**: Building Evolutionary Architectures 2nd Ed 第7章 p.150

### 4. アジリティ等の測定不能な複合的特性を、サイクルタイムなど測定可能な下位特性に分解し、それ自体をfitness functionとして監視しているか
- **なぜ重要か**: 進化速度はサイクルタイムに比例する(v∝c)ため、サイクルタイムの劣化が進化阻害の早期警告指標になる
- **出典**: Building Evolutionary Architectures 2nd Ed 第8章 p.187; Software Architecture: The Hard Parts 第1章 p.7

### 5. 自動化できないfitness function(法務レビュー等の手動検証)も、できる限り頻度を上げてパイプラインに組み込んでいるか
- **なぜ重要か**: 手動検証を放置すると変化の検知が手遅れになりやすいため
- **出典**: Software Architecture: The Hard Parts 第1章 p.11

### 6. 進化的アーキテクチャのfitness functionを整備しているが、それが人同士の対話・合意形成の代替にはならないと認識し、ガバナンス上の議論の場を別途確保しているか(fitness functionを増やすほど良いという一方向の前提に陥っていないか)
- **なぜ重要か**: fitness functionは『測れるもの』しか捉えられず、本来必要な人同士の対話を置き換える道具ではないため
- **出典**: Building Microservices 2nd Ed 第16章 p.539-540

### 7. Strangler Fig/Fidelity fitness functionで新旧実装を本番環境で並行稼働させ、結果の忠実性(fidelity)を自動検証しているか
- **なぜ重要か**: 本番トラフィックでの並行比較により、リスクを抑えつつ置き換えの正しさを検証できるため(GitHubのScientistフレームワークの実例)
- **出典**: Building Evolutionary Architectures 2nd Ed 第4章 p.76-78, 第9章 p.215

### 8. ADRに『何を・なぜ決めたか』だけでなく、その決定をどのfitness functionで継続的に統治するかを記載しているか
- **なぜ重要か**: 『測定できるものだけが統治できる』というテーゼの通り、検証手段を併記しないと決定は形骸化するため
- **出典**: Building Evolutionary Architectures 2nd Ed 第4章 p.75, 79

### 9. ADRのOptionsに2つ以上(目安3-5)の選択肢を挙げ、採用案だけでなく却下案についても『selected/rejected because/despite』の形で理由を明記しているか
- **なぜ重要か**: 選択肢が1つしかないADRは『想像力の欠如』のサインであり、却下理由を残さないと後から同じ議論が繰り返されるため
- **出典**: Facilitating Software Architecture 第6章 p.121-124, 131-132, 146-148

### 10. ADRにライフサイクル(ステータス遷移)を定め、確定(Accepted)後の記録は不変として扱い、変更が必要な場合は旧記録を保持したまま新しいADRへのリンクで改訂するプロセスを運用しているか(例: Facilitating Software ArchitectureのDraft→Proposed→Accepted→Supersededモデル)
- **なぜ重要か**: ADRを事後に書き換えると決定の履歴・経緯が追跡できなくなり、組織学習と監査可能性が機能しなくなるため
- **出典**: Facilitating Software Architecture 第6章 p.150-153; Fundamentals of Enterprise Architecture 第14章 p.224-225(同書独自のDraft→In progress→Review→Approved→Deprecatedライフサイクルで同様の原則を提示)

### 11. その決定がOne-way door(やり直し困難)かTwo-way door(変更容易)かを事前に見極め、前者にはより慎重な検討を重点的に配分しているか
- **なぜ重要か**: アーキテクチャ能力は有限資源であり、やり直しコストの高い決定ほど重点的に検討すべきため
- **出典**: Fundamentals of Enterprise Architecture 第6章 p.139-140; Mastering API Architecture 第10章 p.233

### 12. レガシー移行・SaaS化等において、ビッグバンでの一括リライトやゼロからの作り直し(Second System)を避け、Strangler Figでコアロジックやストレージ層のような中核コンポーネントを抽象化しながら新旧実装を並行稼働させ、段階的な進化ステップ(例: リフト&シフト→レイヤード→サービス単位)として移行を計画しているか
- **なぜ重要か**: ゼロからの作り直しは常に本番追従の機能開発に追われ『いつまでも完成しない』罠(Second System trap)に陥りやすく、調査した移行パターンのいずれもビッグバン移行を推奨していないため
- **出典**: Architecture Patterns with Python エピローグ p.218-220; Designing Distributed Systems 2nd Ed 第16章 p.188-189; Mastering API Architecture 第8章 p.203-205; Building Multi-Tenant SaaS Architectures 第13章 p.344, 357-358

### 13. Strangler Fig的に新旧システムを並行稼働させる移行期間中、両者間のデータ整合性を維持する仕組み(プロキシ層でのビジネスロジック混入回避含む)を設計しているか
- **なぜ重要か**: 移行期の最大の難所は新旧データのコヒーレンシ確保であるため
- **出典**: Mastering API Architecture 第8章 p.204

### 14. feature toggleやblue/greenデプロイ、canary release、parallel runで『デプロイ』と『リリース』を分離し、変更の影響範囲・可逆性を確保しているか
- **なぜ重要か**: 進化的変更は失敗を前提とすべきで、ロールバック容易性が安全網になるため
- **出典**: Building Microservices 2nd Ed 第8章 p.269-273; Building Evolutionary Architectures 2nd Ed 第7章 p.164

### 15. 目的を終えたフィーチャートグルやコードパスを定期的に洗い出し、積極的に除去するプロセスがあるか
- **なぜ重要か**: 使い回されたフィーチャートグルがSMARS/PowerPegで45分で4億ドル、Knight Capitalで4.6億ドルの損失を招いた実例があるため
- **出典**: Building Evolutionary Architectures 2nd Ed 第7章 p.163-164; Mastering API Architecture 第5章 p.128

### 16. データベーススキーマの破壊的変更にExpand/Contractパターンを用い、移行期間中は新旧スキーマの両方をサポートしているか
- **なぜ重要か**: 統合先(コンシューマ)を壊さずに段階的に移行できるため
- **出典**: Building Evolutionary Architectures 2nd Ed 第6章 p.128-132

### 17. アンチコラプションレイヤーの導入を『最後の責任ある瞬間(last responsible moment)』まで待ち、早すぎる抽象化を避けているか
- **なぜ重要か**: 早すぎる抽象化も遅すぎる密結合も、どちらも将来の進化を阻害するため
- **出典**: Building Evolutionary Architectures 2nd Ed 第7章 p.165-167

### 18. 大規模構造(large-scale structure)を事前に硬直的に決めず、アプリケーションの成長に合わせて進化させているか。構造を適用する際は『包括性』より『最小限で効果のあるセット』を優先しているか
- **なぜ重要か**: 過度な上流設計は開発者の裁量を奪い形骸化や反逆を招き、不適合な構造は無いよりも悪いため
- **出典**: DDD Reference VI章 Evolving Order, p.48

### 19. ローリングアップデートを採用する前に、state store・スキーマ・内部トポロジーへの破壊的変更が紛れ込んでいないか確認しているか
- **なぜ重要か**: これが『最もよくある間違い』として名指しされているため
- **出典**: Building Event-Driven Microservices 2nd Ed 第21章 p.430

### 20. デプロイアーティファクトを環境ごとに再ビルドせず、一度だけビルドしたバイナリを全環境へ同一のまま昇格させる(radical immutability)運用になっているか
- **なぜ重要か**: 環境間での再ビルドは『it works on my machine』問題の根本原因であり、fitness functionの検証結果がビルドごとに変わらないという前提を支えるため
- **出典**: Semantic Software Design 第7章 p.173-174, 第12章 p.289; Building Microservices 2nd Ed 第7章 p.204-205

### 21. 本番環境で意図的に障害を起こす実践(Chaos Monkey等のカオスエンジニアリング)を、継続的なレジリエンシー検証として取り入れているか
- **なぜ重要か**: 障害注入による継続検証は、本番相当の条件でレジリエンシーを実証する実践的なfitness functionの一種であるため
- **出典**: Semantic Software Design 第3章 p.63, 第12章 p.290

### 22. protobuf等のメッセージスキーマの進化ルール(フィールド追加は非破壊、削除はdeprecated化+番号reserved)を明文化し、後方互換性だけでなく前方互換性(新旧producer/consumer混在)も設計時に考慮しているか
- **なぜ重要か**: ルール違反は互換性破壊を招き、メッセージング系では新旧複数バージョンが同時稼働しうるため
- **出典**: Learning API Styles 第8章 p.247, 第2章 p.47

### 23. APIのDeprecationとRetirementを区別し、Sunset/Deprecationヘッダー(RFC 8594)等で廃止予定を明示するプロセスを運用しているか
- **なぜ重要か**: 曖昧な廃止プロセスは、誰にも使われなくなっても残り続ける『Zombie APIs』を生むため
- **出典**: Learning API Styles 第1章

### 24. 影響が大きい決定を、機能的フラクチャー面・タイミング・作業ブロック解除等の観点でできる限り小さく分割し、『ウォーキングスケルトン』(自動ビルド・デプロイ・テストが通る最小機能スライス)で最初期から検証しているか
- **なぜ重要か**: 小さく分割し早期に検証することで、大きな決定が失敗した際のコストを下げられるため
- **出典**: Facilitating Software Architecture 第13章 p.333-342

### 25. 重要な決定を『本番投入まで検証されない仮説』として扱い、Fast and wrong is better than slow and correctの精神で早く本番フィードバックを得ているか
- **なぜ重要か**: 失敗を恐れて決定を遅らせることこそが最大のリスクであるため
- **出典**: Facilitating Software Architecture 第13章 p.345-346

### 26. 技術的負債の解消を単独のアーキテクチャ提案として予算化せず、新機能開発プロジェクトに紐づけて正当化(architecture tax)しているか
- **なぜ重要か**: ビジネスは機能価値への投資を優先したがるため、単独提案では予算が付きにくい
- **出典**: Architecture Patterns with Python エピローグ p.212

### 27. フレームワーク更新は積極的(push)に、ライブラリ更新は必要になった時点(pull)にと更新ポリシーを明確に区別し、同時サポートバージョン数を2つ程度に制限しているか
- **なぜ重要か**: フレームワークは結合度が高く2メジャーバージョン以上の遅延が致命的になり、サポートバージョン数が増えるほどテスト・運用負荷が指数的に増えるため
- **出典**: Building Evolutionary Architectures 2nd Ed 第7章 p.170-172

### 28. 外部OSS依存の消失・破壊的変更に備え、内部リポジトリでのpullモデルの依存管理をしているか
- **なぜ重要か**: left-pad事件のように、単一の小さな依存が消失するだけでエコシステム全体が壊れうるため
- **出典**: Building Evolutionary Architectures 2nd Ed 第7章 p.169-170

### 29. 技術標準にChampion/Challengerモデル(現行標準への挑戦者を育成・評価する仕組み)を組み込み、標準が陳腐化(技術的負債化)するのを防いでいるか
- **なぜ重要か**: 一度宣言した標準を更新しないと技術的陳腐化と人材流出を招くため(『Suffocation』の失敗例)
- **出典**: Fundamentals of Enterprise Architecture 第5章 p.124, 135

### 30. データマイグレーション(既存データの書き換え)より、状態ストアの再構築(rebuild)を優先する移行方針になっているか
- **なぜ重要か**: データマイグレーションは危険で不整合の温床になると明記されているため
- **出典**: Building Event-Driven Microservices 2nd Ed 第8章 p.189

### 31. コードチャーン分析や複雑度計測ツールで『変更のレバレッジポイント』を特定した上で刷新・リファクタリングに着手しているか(勘や評判だけで着手していないか)
- **なぜ重要か**: 技術的負債の解消優先順位を客観的根拠に基づいて決めることで、効果の薄い刷新に労力を割くのを避けられるため
- **出典**: Mastering API Architecture 第8章 p.202

### 32. メダリオンアーキテクチャ等の階層構造・内部構造(レイヤー数、データプロダクト層の追加等)や、それを統治するガバナンスモデル自体を、組織の成熟度や要件変化に応じて増減・進化させられる『適応度』を持たせているか(一度に完成させず段階的に進化させる計画があるか)
- **なぜ重要か**: 『3層構成が常に正解』とは限らず、ガバナンスも中央集権→コミュニティ型→連合型のように段階的に進化させた実例があるため
- **出典**: Building Medallion Architectures 第11章 p.305-312, 第9章 p.280, 第10章 p.284-287

### 33. レガシー移行を単純な『リフト&シフト』で終わらせず、抽象化レイヤーを介してStrangler Fig的に部品を段階的に置き換えながら、クラウドネイティブ・疎結合・イベント駆動の利点を探索する設計になっているか
- **なぜ重要か**: 線形発想の移行はクラウドネイティブの利点を活かせず、抽象化レイヤー越しの段階的置換の方がモダナイズの機会を活かせるため
- **出典**: Learning Systems Thinking p.194-195

### 34. fitness functionによる継続検証に加えて、段階的成熟度モデル(例: Green Software Maturity Matrix)で現状レベルと次に取るべきアクションを定期的に自己評価する仕組みも持っているか
- **なぜ重要か**: 詳細な測定データが揃っていない段階でも、成熟度レベルを1段階進める確実な次の一手を特定できるため
- **出典**: Building Green Software 第12章 p.184-185
