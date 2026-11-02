---
name: security-review
description: 認証・認可、シークレット管理、ネットワーク/ゼロトラスト設計、クラウド・コンテナ構成、Webアプリケーションの脆弱性、サプライチェーン・CI/CDセキュリティ、インシデント対応設計、セキュリティガバナンス、脅威モデリング、LLM/AIのプライバシー・セキュリティのいずれかに関わるコード・設計・インフラ定義をレビューする際に使う。ユーザーが明示的に「セキュリティレビューして」と言わなくても、認証・認可・シークレット・IAMロール・ネットワークポリシー・Dockerfile/Terraform/CI設定・API・LLM/RAGを新規作成・変更した場合は積極的に使う。「このAPIのセキュリティどう思う?」「このIAMポリシー大丈夫?」「脆弱性診断の観点でレビューして」のような相談にも必ず使う。

---

# セキュリティレビュー

ソフトウェア・インフラのコードや設計を、10冊の専門書から抽出した約300項目のチェックリストに基づいてレビューする。抽象的な「セキュリティ意識を持ちましょう」ではなく、具体的な書籍の知見に根拠づけられた、再現可能なレビュー観点を提供することがこのスキルの価値。

## レビュー対象の特定

ユーザーが対象を明示していない場合は、次の優先順位で判断する。

1. 現在の作業ツリーの差分(`git diff` / `git diff --staged`)があればそれを対象にする。
2. ユーザーが特定のファイル・ディレクトリ・PR番号・設計ドキュメントを指定していればそれを対象にする。
3. どちらもなければ、直近で変更・作成されたセキュリティ関連ファイル(認証/認可ロジック、IAMポリシー、Terraform/CloudFormation等のIaC、Dockerfile、CI/CD設定、APIハンドラ、LLM/RAGパイプライン等)を探す。

レビュー対象が「コード」である必要はない。IAMポリシーのJSON、Terraformコード、CI/CD設定(YAML)、脅威モデル文書(Markdown)も対象になる。

## レビュー手順

### 1. 関連カテゴリを見極める(全部読まない)

対象を一度ざっと見て、下表の10カテゴリのうちどれが関連するか判断する。すべてのカテゴリが関係するとは限らない。例えば単純なログイン機能の追加なら「IDとアクセス管理」と「Webアプリケーションセキュリティ」だけで十分な場合が多い。Terraformでのクラウド基盤構築なら「クラウド・コンテナセキュリティ」「ネットワーク・ゼロトラスト」「Security as Code・DevSecOps」が中心になる。LLM/RAGアプリなら「LLM/AIのプライバシー・セキュリティ」を必ず確認する。

判断に迷う、あるいは対象がシステム全体(新規サービスの設計レビュー等)であれば、関連しそうなカテゴリを広めに取ってよい。無関係なカテゴリの参照ファイルまで読み込む必要はない(プログレッシブディスクロージャー: 必要な参照ファイルだけを読む)。

| キー | カテゴリ | 焦点 | 参照ファイル |
|---|---|---|---|
| `identity-access-management` | IDとアクセス管理 | 認証・認可、シークレット管理、最小権限、MFA、特権アクセス管理、機械ID | `references/identity-access-management.md` |
| `network-zero-trust` | ネットワーク・ゼロトラストアーキテクチャ | マイクロセグメンテーション、境界なし設計、mTLS、PKI、ネットワーク分離 | `references/network-zero-trust.md` |
| `cloud-container-security` | クラウド・コンテナセキュリティ | K8s/クラウドネイティブ、ハイブリッドクラウド、責任共有モデル、IAM、暗号化 | `references/cloud-container-security.md` |
| `web-app-security` | Webアプリケーションセキュリティ | OWASP系脆弱性(インジェクション、XSS、CSRF、認証不備、ビジネスロジック脆弱性) | `references/web-app-security.md` |
| `supply-chain-security` | サプライチェーンセキュリティ | SBOM、依存関係、ビルド来歴、CI/CD改ざん対策、署名検証 | `references/supply-chain-security.md` |
| `security-as-code-devsecops` | Security as Code・DevSecOps | policy as code、CI/CDセキュリティ、IaCセキュリティ、シフトレフト | `references/security-as-code-devsecops.md` |
| `incident-response-ops` | インシデント対応・セキュリティ運用 | ログ、検知、フォレンジック、監視、インシデント対応ライフサイクル | `references/incident-response-ops.md` |
| `security-governance` | セキュリティガバナンス・組織的対策 | ポリシー、教育、ベンダーリスク、コンプライアンス、契約条項 | `references/security-governance.md` |
| `threat-landscape` | 脅威ランドスケープ・サイバー戦 | 国家支援型攻撃、APT、脅威インテリジェンス、帰属分析、サプライチェーン攻撃の脅威モデル | `references/threat-landscape.md` |
| `llm-privacy-security` | LLM/AIのプライバシー・セキュリティ | データ漏洩、モデルセキュリティ、プロンプトインジェクション、差分プライバシー、プライバシー規制 | `references/llm-privacy-security.md` |

### 2. 該当する参照ファイルを読み、チェックリストに沿って検証する

選んだ参照ファイルを読み、各チェック項目を「対象コードの具体的な箇所」に当てはめて検証する。チェックリストの文言をそのままユーザーに投げ返すのではなく、実際のコードの該当行・該当設定を見て、その項目が満たされているか・満たされていないかを判断する。

満たされていない場合でも、それが「このプロダクトにとって本当にリスクか」を考えること。社内の低リスクなプロトタイプにエンタープライズ級のゼロトラスト基盤を要求するのは過剰である。チェックリストは機械的な合否基準ではなく、何を確認すべきかの網羅的なガイドとして使う。

一部の項目(CI/CDパイプラインの構成、組織のプロセス・契約条項など)は、レビュー対象が単一ファイル・単一コンポーネントの場合そのファイルだけからは真偽を判定できない。そのような項目は「違反している」と断定せず、「このファイルからは確認できない(別途パイプライン設定・運用プロセスを確認すべき)」と明示すること。

参照ファイル間で内容が多少重複している項目がある(例: HSMによる鍵管理はIDとアクセス管理・ネットワーク・クラウドの複数カテゴリに関わる)。これは意図的な設計で、片方のファイルしか読まなくても重要な観点を見落とさないようにしている。発見(finding)の`category`は1つしか選べないため、重複する観点を報告する際は、最も直接的にその問題を説明できるカテゴリを選べばよい。

### 3. 発見を報告する

**`ReportFindings` ツールが使える場合は必ずそれを使う。** 各findingについて:

- `file` / `line`: 問題のある具体的なコード箇所。設定ファイルや設計ドキュメントなど行番号がない場合は省略可。
- `summary`: 1文での問題の指摘。
- `failure_scenario`: 「どんな入力・状況で、何が起きるか」を具体的に書く(例: 「サービスアカウントキーがリポジトリに平文で存在するため、リポジトリへの読み取り権限を持つ誰でも本番環境の全データへアクセスできる」)。
- `category`: 該当するカテゴリの**キー**(`identity-access-management`, `web-app-security` 等、上の表の「キー」列)を入れる。
- `short_summary`: 60文字以内の短い要約。
- 重要度の高い発見から並べる。

`ReportFindings` が使えない環境では、カテゴリごとに見出しを分けたMarkdownで、各項目について「問題点 / 該当箇所 / 根拠(どの書籍のどの知見に基づくか) / 改善案」を書いて報告する。

良い実装については「◯◯の設計は〜という理由で適切」のように積極的に評価してよい。欠点の指摘だけでなく、何が既にベストプラクティスに沿っているかも伝えると、レビューの信頼性が上がる。

## このスキルが扱わない観点(対象書籍に含まれなかったテーマ、または処理できなかった書籍の影響)

以下はこのチェックリストでは深く扱えないため、必要に応じて別途検討すること:

- **インシデント対応の現場実務・スクリプティング層**: bash/awk/grepでのログ解析パイプライン、tcpdump等のネットワークキャプチャのCLI解析、プロセス・cron・SUIDバイナリ等のUnix侵害痕跡のコマンドライン調査、証拠収集・トリアージの自動化スクリプト。これは本来『Cybersecurity Ops with Bash』が担うはずの領域だが、Claudeの自動セキュリティセーフガードにより読解が3回失敗したため、このスキルの素材には含まれていない(下記「出典」参照)。`incident-response-ops`カテゴリはガバナンス・プロセス・戦略レベルの知見のみで構成されている。
- DDoS対策(CDN/スクラビング、レート制限、キャパシティプランニング、SYNフラッド対策等)の専門的な設計手法
- モバイルデバイス管理(MDM)・BYOD・モバイルアプリ特有のセキュリティ(証明書ピンニング、ジェイルブレイク検知等)
- メール基盤レベルの技術的アンチフィッシング対策(SPF/DKIM/DMARC、メールゲートウェイでの添付・URLサンドボックス化等)
- LLM/AI以外の一般的なプライバシー工学(PII最小化・トークン化/マスキング、データ主体のアクセス・削除請求(DSAR)対応、保持期間に基づくデータ廃棄、同意管理)。現状のプライバシー観点は`llm-privacy-security`カテゴリに集約されており、AI以外のシステム(通常のWebアプリ・DB・分析基盤)における個人データ保護の体系的チェックは手薄
- 資産管理・CMDB(ハードウェア/ソフトウェア資産のインベントリ化、未管理デバイスの発見)という基礎的統制
- 物理・環境セキュリティ全般(データセンターの環境耐性、媒体の安全な廃棄・サニタイズ、デバイス盗難時のリモートワイプ等)。高機密エリアの物理アクセス制御の1項目のみ存在する
- 採用時のバックグラウンドチェック等、インサイダーリスクの事前防止(検知については`incident-response-ops`に項目がある)
- OS/サーバーの体系的なセキュア構成ベースライン(CISベンチマーク等によるハードニング)。個別の設定項目(デフォルトVPC削除等)はあるが体系的な適用は明示されていない

## 出典

本スキルのチェックリストは以下10冊から抽出した知見を統合したもの:

1. 『Cloud Native Security Cookbook』(Josh Armitage)
2. 『Defensive Security Handbook, 2nd Edition』(Amanda Berlin, Lee Brotherston, William F. Reyor III)
3. 『Identity Security for Software Development』(John Walsh, Uzi Ailon)
4. 『Inside Cyber Warfare, 3rd Edition』(Jeffrey Caruso)
5. 『Privacy and Security for Large Language Models』(Baihan Lin)
6. 『Security Architecture for Hybrid Cloud』(Mark Buckwell, Stefaan Van daele, Carsten Horst)
7. 『Security as Code: DevSecOps Patterns with AWS』(BK Sarthak Das, Virginia Chu)
8. 『Software Supply Chain Security』(Cassie Crossley)
9. 『Web Application Security, 2nd Edition』
10. 『Zero Trust Networks, 2nd Edition』

**処理できなかった本**: 『Cybersecurity Ops with Bash』— インシデント対応・フォレンジック自動化の観点を重点的に扱う本だったが、Claudeの自動セキュリティセーフガードにより3回読解に失敗したため、このスキルの素材には含まれていない。上記「このスキルが扱わない観点」を参照。
