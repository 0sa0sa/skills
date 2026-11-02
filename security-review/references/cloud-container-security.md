# クラウド・コンテナセキュリティ

K8s/クラウドネイティブ、ハイブリッドクラウド、責任共有モデル、IAM、暗号化に関わる設計・実装を見る。

## チェックリスト

### 1. クラウドの責任共有モデル(Shared Responsibility Model)を正しく理解し、IaaS/PaaS/SaaSごとに自組織が担うべき設定(ストレージの公開設定、IAM設定等)に漏れがないか文書化・検証しているか
- **なぜ重要か**: 「誰が何を守るか」の誤解は重大なセキュリティギャップを生む。責任範囲の誤解がインシデントの土台になることが繰り返し指摘されている。
- **出典**: Defensive Security Handbook 2nd Ed 14章(p.151,159); Identity Security for Software Development 4章・6章; Security Architecture for Hybrid Cloud 7章・11章; Software Supply Chain Security 6章(p.94, Control CLD-01)

### 2. クラウド設定の誤り(過剰公開、過剰権限等)をCSPM(AWS Config/GCP Security Command Center等)で継続的に監査する仕組みがあるか
- **なぜ重要か**: Capital One 2019侵害(SSRF+過剰権限、1億人超流出)は誤設定が発端であり、継続監査なしでは発見が遅れる。
- **出典**: Defensive Security Handbook 2nd Ed 14章(p.152-153)

### 3. クラウドストレージの既定非公開設定・暗号化を自動検出・強制しているか。クラウドAPI自体は「Secure by Default」ではない点を理解し、IaCテンプレート側で暗号化・非公開を明示的に強制しているか。また暗号化(KMS等)は非遡及的であり適用前に書き込まれたデータは平文のまま残る点も認識しているか
- **なぜ重要か**: 意図しないパブリックデータ格納は最多のデータ漏洩原因と明記されており、Microsoftも2022年2.4TB・2023年38TBの設定ミスによる漏洩を2年連続で起こしている。
- **出典**: Cloud Native Security Cookbook 第7章7.11(p.398); Security Architecture for Hybrid Cloud 第7章; Software Supply Chain Security 第7章(p.119); Security as Code(p.8, 23, 43)

### 4. 自動修復(auto-remediation)の適用範囲を低運用リスク・高セキュリティリスク対象に限定し、IaC管理下のリソースには適用していないか(検出→防止→修復の順で投資し、修復への依存を最終手段としているか)
- **なぜ重要か**: 自動修復はコードで定義された状態との構成ドリフトを生みIaCの価値を損なう。検出・防止より修復に依存するとコンプライアンス債務が増大する。
- **出典**: Cloud Native Security Cookbook 第7章7.10-7.12(p.394-395, 399, 405)

### 5. サービスアカウント/マネージドIDの権限昇格ベクトル(getAccessToken, actAs, setIamPolicy等)を特定し厳格に制限し、サービスアカウントキーの作成・アップロードを組織ポリシーでデフォルト無効化しているか
- **なぜ重要か**: サービスアカウント乗っ取りは権限昇格・横移動の主要経路であり、漏洩キーは永続的な直接攻撃経路になる。
- **出典**: Cloud Native Security Cookbook 第8章8.1(p.411)

### 6. AWSでPermission Boundary/SCPを使い、Azureでは「User Access Administrator」の自己昇格をAzure Policyで防止するなど、プリンシパルが自分より高権限の別プリンシパルを作成・自己昇格できないよう制限しているか
- **なぜ重要か**: 境界がなければ権限昇格を防げず、Azureの既定動作では管理者ロール=実質Owner権限になってしまう。
- **出典**: Cloud Native Security Cookbook 第8章8.2-8.3(p.415, 419-420)

### 7. 恒久的な特権付与を避け、Just-In-Time/時間制限/承認制の特権アクセス管理(Azure PIM等)やゼロ・スタンディング・プリビレッジ(ZSP)+TEA方式を採用しているか
- **なぜ重要か**: 恒久的特権は漏洩時の影響範囲・期間を増大させ、クラウドの動的性質上「常にオンの権限」は管理困難で攻撃対象を広げる。
- **出典**: Cloud Native Security Cookbook 第8章8.3(p.421); Identity Security for Software Development 第6章

### 8. IAMロール等に最小権限の原則を適用し、付与された権限が実際の使用状況と一致しているか、過剰権限(overpermissioned)を継続的に監視・棚卸ししているか
- **なぜ重要か**: 過剰権限はクラウドにおける典型的な誤りであり、定常的な権限レビューがなければ権限の肥大化(permission creep)を検知できない。
- **出典**: Defensive Security Handbook 2nd Ed 14章(p.156)

### 9. 複数CSP(AWS/GCP/Azure)やIdPを併用・移行する際、IAMモデルの構造的違い(JSONポリシーvs RBAC、権限とリソースの結合/分離等)を理解しマッピングを文書化しているか
- **なぜ重要か**: 各CSPのIAM実装は構造が異なり、移行時の見落としが過剰/不足権限を生む。
- **出典**: Identity Security for Software Development 第6章

### 10. IaCコードレビュー(自動スキャン含む)で、パブリックアクセスブロックの無効化やワイルドカードPrincipal(`*`)・ワイルドカードResource等の過剰許可設定を検出しているか
- **なぜ重要か**: S3バケットのTerraform例ではblock_public_acls=false等の設定が実際の情報漏洩パターンと一致する。
- **出典**: Identity Security for Software Development 第6章; Security as Code(p.65-66)

### 11. Kubernetesクラスタのネットワークでデフォルト拒否(default-deny)のNetworkPolicyを設定し、明示的に許可した通信のみを通しているか
- **なぜ重要か**: Kubernetesはデフォルトでトラフィックが無制限であり、何も設定しないと全通信が許可されてしまう。
- **出典**: Identity Security for Software Development 第7章

### 12. サービスメッシュ導入時にmTLS(相互TLS)を強制し、Pod間通信をゼロトラストの原則(default-deny→必要な通信のみ許可)で構成しているか
- **なぜ重要か**: 証明書ベースのID確認によりPodのなりすまし(impersonation)攻撃を防止できる。
- **出典**: Identity Security for Software Development 第7章; Security Architecture for Hybrid Cloud 第8章

### 13. コンテナ/Podをroot権限で実行せず、runAsNonRoot・privileged:false・allowPrivilegeEscalation:falseなど最小権限のセキュリティコンテキストで実行しているか
- **なぜ重要か**: 特権コンテナはホストのカーネル機能・デバイスへの全面アクセスを許してしまう。
- **出典**: Identity Security for Software Development 第7章; Privacy and Security for Large Language Models 第5章

### 14. コンテナイメージを最小化(slim/distroless)し、latestタグでなく固定バージョン+ダイジェスト検証を使用した上で、ベースイメージ・依存関係・設定の全層に対し定期的な脆弱性スキャンを実施しているか
- **なぜ重要か**: distrolessやAlpineは攻撃対象領域を削減し、固定タグは予測不能な更新によるリスクを防ぐ。
- **出典**: Identity Security for Software Development 第7章; Privacy and Security for Large Language Models 第5章

### 15. Kubernetes Secretsをbase64エンコードのまま使わず、外部シークレット管理ツール(External Secrets Operator等)と連携しているか
- **なぜ重要か**: base64はエンコードであり暗号化ではなく、設定ファイルの閲覧で容易に読めてしまう。
- **出典**: Identity Security for Software Development 第7章

### 16. コンテナに読み取り専用ファイルシステムとCPU/メモリ/IOのリソース上限を設定し、Pod Security PolicyとNetwork Policyでオーケストレーション層の通信経路・Secret管理を制御しているか
- **なぜ重要か**: サプライチェーン経由の改ざんやDoSを防止するための基本的なコンテナセキュリティ設定である。
- **出典**: Privacy and Security for Large Language Models 第5章

### 17. EKSクラスタ/コンテナワークロードに対しカオスエンジニアリング(Chaos Mesh/AWS FIS)でCPUストレスや障害注入実験を行い、障害時のセキュリティ挙動(オートスケール・アラートの実効性等)を検証しているか
- **なぜ重要か**: 分散システムの可用性劣化は検知漏れになりやすく、障害注入実験によって事前にオートスケール・アラートの実効性を検証できる。
- **出典**: Security as Code(p.78-79, 78-84)

### 18. 各ディスク/DB/ストレージの暗号化に単一の共有キーではなく用途別の顧客管理キー(CMK)/BYOK(Bring Your Own Key)を適用し、HSMによる鍵管理で特権を持つプロバイダ従業員からもデータを保護できているか
- **なぜ重要か**: 共有キーは単一障害点・単一侵害点となりブラストラディアスを拡大する。
- **出典**: Cloud Native Security Cookbook 第4章4.1-4.3(p.127,133,141); Security Architecture for Hybrid Cloud 第8章

### 19. 機密データの所在を自動発見・分類するDLPツールを全社規模で中央集約運用しているか
- **なぜ重要か**: 数百チーム規模では人手でデータの所在を追い続けることは不可能であり、分類の抜け漏れが保護対象の見落としにつながる。
- **出典**: Cloud Native Security Cookbook 第4章冒頭/4.10(p.123, 169)

### 20. CASB(クラウドアクセスセキュリティブローカー)を導入し、シャドーITの可視化とDLPを行っているか
- **なぜ重要か**: 許可されていないクラウドサービス利用(シャドーIT)は組織の可視性の外でデータ漏洩経路になる。
- **出典**: Zero Trust Networks 2nd Ed 第12章(p.284-285)・第8章(p.186-187)

### 21. 内部サービス間通信をPrivateLink/Private Endpoint等のプライベート経路で提供する場合でも、「プライベートだから安全」と転送時暗号化(TLS)を省略せず別途設計しているか
- **なぜ重要か**: プライベート経路であっても転送時暗号化を省略してはならない。
- **出典**: Cloud Native Security Cookbook 第5章5.13(p.262)

### 22. 鍵管理・ネットワーク機器間で「鍵を車に閉じ込める」循環依存(ファイアウォール再起動にHSM復号が必要、HSM復号にネットワーク接続が必要…)が発生しないか可用性/リカバリ設計で検証しているか
- **なぜ重要か**: この種の循環依存はDR設計上の罠であり、秘密管理・証明書管理でも同様の問題が起きる。
- **出典**: Security Architecture for Hybrid Cloud 第4章

### 23. 機密性の高いモデル・暗号鍵・データを扱う場合、HSM・Secure Boot・メモリ暗号化(AMD SEV/Intel TME)や機密コンピューティング(TEE)の導入を検討し、パブリッククラウド事業者そのものを暗黙に信頼していないか
- **なぜ重要か**: 物理アクセス攻撃・プロバイダ従業員からモデル重みやデータという知的財産を保護するため。
- **出典**: Privacy and Security for Large Language Models 第5章; Zero Trust Networks 2nd Ed 第5章(p.87-88)・第7章(p.161-163)

### 24. 本番/検証/開発環境をアカウント・プロジェクト単位で分離し、リージョンロックを組織階層の最上位で設定し、AWS組織のmanagementアカウントへのワークロード配置を最小化しているか(SCPがmanagementアカウントには適用されないため)
- **なぜ重要か**: 一律のアクセスレベルは開発速度かテスト実効性のどちらかを損なう。SCPが効かないmanagementアカウントにワークロードを置くとガードレールの穴になる。
- **出典**: Cloud Native Security Cookbook 第2章2.1-2.6(p.24-25, 33-34, 38-39, 42, 46, 48)

### 25. マルチクラウド/ハイブリッドクラウドでCSPごとに異なる用語・ネットワーク設計パターン(AWS/Azure/GCP/IBM Cloud)の違いを理解し、ランディングゾーン(原則・ポリシー・実践・エンタープライズパターン)を整備した上で一貫した設計アプローチを確立しているか
- **なぜ重要か**: ネットワーク層では「one-size-fits-all」は存在せず、個別プロジェクトごとにガバナンス基盤を再構築するのは非効率で統制が効きにくい。
- **出典**: Security Architecture for Hybrid Cloud 第7章・第8章

### 26. クラウド版可用性ゾーン(マルチゾーンリージョン等)を採用する際、オンプレミスのDR前提(同期レプリケーション等)がそのまま成立しないこと(レイテンシ・帯域制約)を検証しているか
- **なぜ重要か**: オンプレミス時代のDR設計の前提がクラウドではそのまま通用せず、セキュリティサービスの可用性にも影響する。
- **出典**: Security Architecture for Hybrid Cloud 第7章・第8章

### 27. クラウドモデル(SaaS/IaaS/PaaS/コンテナ/サーバーレス)ごとに攻撃対象面・必要な統制が異なることを理解し、個別にセキュリティ要件を設計しているか
- **なぜ重要か**: 攻撃対象面と必要な統制が種類ごとに異なるため、一律のチェックリストでは不十分。
- **出典**: Software Supply Chain Security 第6章 Table6-2(p.99-100)

### 28. VMフリート全体の棚卸しとパッチ適用をクラウド各社のマネージドサービス(VM Manager/SSM/Azure Automation)で自動化し、適用時にはdisruption budget・事前/事後フック・カスタムベースラインで業務影響を抑えているか
- **なぜ重要か**: 動的に増減するクラウドリソースの脆弱性管理は手作業では対応できず、一括パッチ適用は本番システムに過大な障害リスクを与える。
- **出典**: Cloud Native Security Cookbook 第8章8.4-8.9(p.423, 427, 437, 441, 445)

### 29. API・環境・接続全てにアクセス制御とログ・監視を実装しているか
- **なぜ重要か**: T-Mobile(2022年、3,700万件流出、API認証不備が疑われる)のような、認証・監視が不十分なAPIが大規模侵害の起点になる実例がある。
- **出典**: Software Supply Chain Security 第6章(p.104-107)

### 30. クラウドベンダーの第三者認証(FedRAMP等)や自社内部レビューが実運用のセキュリティ実態を正しく反映しているか、独自に検証しているか
- **なぜ重要か**: Microsoft Storm-0558事件では、外部監査・自社内部レビューいずれでも鍵管理の過失が発見されなかった。
- **出典**: Inside Cyber Warfare 3rd Ed 第3章(p.36)

### 31. イミュータブルインフラの原則を適用し、本番環境への直接変更を禁止・検知しているか
- **なぜ重要か**: 本番直接変更は構成ドリフトと不可視な侵害経路を生む。
- **出典**: Software Supply Chain Security 第6章(p.106, Control CLD-06)
