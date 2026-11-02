---
name: software-design-review
description: ソフトウェアアーキテクチャ・設計のコードやドキュメントをレビューする際に使う。アーキテクチャスタイル選定(モノリス/マイクロサービス/イベント駆動)、DDD・ドメインモデリング、分散データ・一貫性(サガ、シャーディング、分散ロック)、API設計、マルチテナント・SaaS設計、進化的アーキテクチャ・ADR、データアーキテクチャ(レイクハウス/メダリオン)、エンタープライズアーキテクチャ・組織設計、サステナビリティ、アーキテクチャ策定プロセスのいずれかに関わるコード・設計ドキュメントを見るときは必ず使うこと。ユーザーが明示的に「レビューして」と言わなくても、サービス分割・API変更・データモデル変更・ADR作成・技術選定の相談があった場合は積極的に使う。「このアーキテクチャどう思う?」「マイクロサービスに分割すべき?」「このAPI設計で大丈夫?」のような相談にも使う。セキュリティ観点のレビューは別スキル `security-review` を使うこと。
---

# ソフトウェア設計レビュー

ソフトウェアアーキテクチャ・設計のコード・ドキュメントを、16冊の専門書(Architecture Patterns with Python、Building Event-Driven Microservices 2nd Ed、Building Evolutionary Architectures 2nd Ed、Building Green Software、Building Medallion Architectures、Building Microservices 2nd Ed、Building Multi-Tenant SaaS Architectures、Designing Distributed Systems 2nd Ed、Facilitating Software Architecture、Fundamentals of Enterprise Architecture、Learning API Styles、Learning Systems Thinking、Mastering API Architecture、Semantic Software Design、Software Architecture: The Hard Parts、Domain-Driven Design Reference)から抽出した約360項目のチェックリストに基づいてレビューする。一般的な「良さそう」という感想ではなく、具体的な書籍の知見に根拠づけられた、再現可能なレビュー観点を提供することがこのスキルの価値。

## レビュー対象の特定

ユーザーが対象を明示していない場合は、次の優先順位で判断する。

1. 現在の作業ツリーの差分(`git diff` / `git diff --staged`)があればそれを対象にする。
2. ユーザーが特定のファイル・ディレクトリ・PR番号・設計ドキュメントを指定していればそれを対象にする。
3. どちらもなければ、直近で変更・作成された設計関連ファイル(アーキテクチャ図、ADR、サービス定義、データモデル、API仕様等)を探す。

レビュー対象が「コード」である必要はない。ADR、アーキテクチャ設計ドキュメント(Markdown)、API仕様(OpenAPI等)、データモデル図の説明も対象になる。

## レビュー手順

### 1. 関連カテゴリを見極める(全部読まない)

対象を一度ざっと見て、下表の10カテゴリのうちどれが関連するか判断する。すべてのカテゴリが関係するとは限らない。例えば単純なAPI変更なら「API設計」だけで十分な場合が多い。マイクロサービスへの分割提案なら「アーキテクチャスタイル選定」「ドメインモデリング・DDD」「分散データ・一貫性」「エンタープライズアーキテクチャ・システム思考」が中心になる。

判断に迷う、あるいは対象がシステム全体(新規アーキテクチャの設計レビュー等)であれば、関連しそうなカテゴリを広めに取ってよい。無関係なカテゴリの参照ファイルまで読み込む必要はない(プログレッシブディスクロージャー: 必要な参照ファイルだけを読む)。

| キー | カテゴリ | 焦点 | 参照ファイル |
|---|---|---|---|
| `architecture-style-selection` | アーキテクチャスタイル選定 | モノリス/マイクロサービス/イベント駆動/ヘキサゴナル等の選定、採用の理由付け、段階的移行 | `references/architecture-style-selection.md` |
| `domain-modeling-ddd` | ドメインモデリング・DDD | 境界づけられたコンテキスト、集約、腐敗防止層、ユビキタス言語、戦略的/戦術的パターン | `references/domain-modeling-ddd.md` |
| `distributed-data-consistency` | 分散データ・一貫性 | CAP定理、サガ、イベントソーシング、分散トランザクション、結果整合性、シャーディング、分散ロック | `references/distributed-data-consistency.md` |
| `api-design` | API設計 | REST/GraphQL/gRPC、バージョニング、契約、破壊的変更管理、BFF | `references/api-design.md` |
| `multi-tenancy-saas` | マルチテナント・SaaSアーキテクチャ | テナント分離、サイロ/プール、ノイジーネイバー対策、ティアリング | `references/multi-tenancy-saas.md` |
| `evolutionary-architecture` | 進化的アーキテクチャ・適応度関数 | フィットネスファンクション、ADR、段階的変更、Strangler Fig、技術的負債 | `references/evolutionary-architecture.md` |
| `data-architecture` | データアーキテクチャ | メダリオン/レイクハウス層構造、データパイプライン、スキーマ管理、キャッシュ | `references/data-architecture.md` |
| `enterprise-systems-thinking` | エンタープライズアーキテクチャ・システム思考 | 組織とアーキテクチャの整合(コンウェイの法則)、ガバナンス、フィードバックループ、全体最適 | `references/enterprise-systems-thinking.md` |
| `sustainability-green-software` | サステナビリティ・グリーンソフトウェア | カーボン効率、運用効率優先、ハードウェア長寿命化、SCI計測 | `references/sustainability-green-software.md` |
| `architecture-facilitation` | アーキテクチャ策定プロセス・ファシリテーション | ADR作成、ワークショップ、助言プロセス、合意形成、トレードオフ分析 | `references/architecture-facilitation.md` |

### 2. 該当する参照ファイルを読み、チェックリストに沿って検証する

選んだ参照ファイルを読み、各チェック項目を「対象コード・ドキュメントの具体的な箇所」に当てはめて検証する。チェックリストの文言をそのままユーザーに投げ返すのではなく、実際の設計・コードの該当箇所を見て、その項目が満たされているか・満たされていないかを判断する。

満たされていない場合でも、それが「このプロダクトにとって本当にリスクか」を考えること。小規模な社内ツールにエンタープライズ級のデータ整合性設計を要求するのは過剰である。チェックリストは機械的な合否基準ではなく、何を確認すべきかの網羅的なガイドとして使う。

一部の項目(組織のガバナンス体制、フィットネスファンクションのパイプライン統合など)は、レビュー対象が単一ファイル・単一コンポーネントの場合そのファイルだけからは真偽を判定できない。そのような項目は「違反している」と断定せず、「このファイルからは確認できない(別途組織・運用プロセスを確認すべき)」と明示すること。

参照ファイル間で内容が多少重複している項目がある(例: 「One-way door/Two-way door」の判断は`architecture-style-selection`と`evolutionary-architecture`の両方に関わる、Dual Write回避は`distributed-data-consistency`と`data-architecture`の両方に関わる)。これは意図的な設計で、片方のファイルしか読まなくても重要な観点を見落とさないようにしている。発見(finding)の`category`は1つしか選べないため、重複する観点を報告する際は、議論の中心がどちらのカテゴリの関心(選定の判断そのものか、一貫性の実装詳細か等)に近いかで選べばよい。どちらでも大差ない場合は、コードの該当箇所がより直接的に説明できる方を選べばよい。

### 3. 発見を報告する

**`ReportFindings` ツールが使える場合は必ずそれを使う。** 各findingについて:

- `file` / `line`: 問題のある具体的なコード・ドキュメントの箇所。行番号がない場合は省略可。
- `summary`: 1文での問題の指摘。
- `failure_scenario`: 「どんな入力・状況で、何が起きるか」を具体的に書く(例: 「複数集約を1トランザクションで更新しているため、サービス分割後に結果整合性への移行が必要になった際、暗黙の整合性前提を持つコードが連鎖的に壊れる」)。
- `category`: 該当するカテゴリの**キー**(`architecture-style-selection`, `domain-modeling-ddd` 等、上の表の「キー」列)を入れる。
- `short_summary`: 60文字以内の短い要約。
- 重要度の高い発見から並べる。バグとは言い切れないが明確に推奨に反する設計判断も、`category` に関連キーを付けて報告してよい(例: 「技術的な理由だけでマイクロサービス化を進めており、独立デプロイ可能性という目標が定義されていない」)。

`ReportFindings` が使えない環境では、カテゴリごとに見出しを分けたMarkdownで、各項目について「問題点 / 該当箇所 / 根拠(どの書籍のどの知見に基づくか) / 改善案」を書いて報告する。

良い設計については「◯◯の設計は〜という理由で適切」のように積極的に評価してよい。欠点の指摘だけでなく、何が既にベストプラクティスに沿っているかも伝えると、レビューの信頼性が上がる。

## このスキルが扱わない観点(書籍に明示的な記載がないか、別スキルの範囲のため含めていない)

16冊の統合過程で、以下の領域は体系的なチェックリストとしては手薄であることが分かった。必要に応じて別途検討・別スキルの利用を:

- **セキュリティ全般**(脅威モデリング、シークレット管理、ゼロトラスト、サプライチェーン等): 姉妹スキル `security-review` を使うこと。本スキルのAPI設計カテゴリにOWASP API Top10等への軽い言及はあるが、体系的なセキュリティレビューはカバーしない。
- **回復性(レジリエンシー)パターンの詳細**: サーキットブレーカー・バルクヘッド・リトライ/バックオフ・タイムアウト予算など、サービス間呼び出しの障害伝播を防ぐ具体的パターンの採用検証は薄い。
- **オブザーバビリティの設計**: 構造化ログ、分散トレーシングの文脈伝播、SLI/SLO/エラーバジェットの設計基準は体系的に扱っていない。
- **災害復旧・事業継続性(DR/BCP)**: RTO/RPO目標、バックアップ/リストア戦略、マルチリージョン構成の検証は含まれない。
- **チームの認知負荷(cognitive load)に基づく境界設計**: コンウェイの法則やチームサイズへの言及はあるが、Team Topologies的な認知負荷ベースの粒度判断は体系化されていない。
- **テスト戦略全体**: 個別パターン(コンシューマ駆動契約テスト等)はあるが、テストピラミッド構成や性能テスト計画全体は扱わない。
- **AI/LLM機能自体の設計品質**: GenAI機能のプロンプト品質・ハルシネーション対策・モデルバージョニング等は扱わない(AIエージェントアプリのレビューは姉妹スキル `ai-agent-review` を使うこと)。
- **キャパシティプランニング・需要予測**: サステナビリティ/マルチテナンシーの文脈で部分的に触れるのみで、非機能要求としての体系的な検証は含まれない。

## 出典

本スキルのチェックリストは以下16冊から抽出した知見を統合したもの:

- 『Architecture Patterns with Python』(Harry Percival, Bob Gregory)
- 『Building Event-Driven Microservices, 2nd Edition』(Adam Bellemare)
- 『Building Evolutionary Architectures, 2nd Edition』(Neal Ford, Rebecca Parsons, Patrick Kua, Pramod Sadalage)
- 『Building Green Software』(Anne Currie, Sarah Hsu, Sara Bergman)
- 『Building Medallion Architectures』(Piethein Strengholt)
- 『Building Microservices, 2nd Edition』(Sam Newman)
- 『Building Multi-Tenant SaaS Architectures』(Tod Golding)
- 『Designing Distributed Systems, 2nd Edition』(Brendan Burns)
- 『Facilitating Software Architecture』(Andrew Harmel-Law)
- 『Fundamentals of Enterprise Architecture』(Tanusree McCabe)
- 『Learning API Styles』(O'Reilly)
- 『Learning Systems Thinking』(Diana Montalion)
- 『Mastering API Architecture』(James Gough, Daniel Bryant, Matthew Auburn)
- 『Semantic Software Design』(Eben Hewitt)
- 『Software Architecture: The Hard Parts』(Neal Ford, Mark Richards, Pramod Sadalage, Zhamak Dehghani)
- 『Domain-Driven Design Reference』(Eric Evans)
