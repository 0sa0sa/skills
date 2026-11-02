# Security as Code・DevSecOps

policy as code、CI/CDセキュリティ、IaCセキュリティ、シフトレフトに関わる設計・実装を見る。

## チェックリスト

### 1. CI/CDパイプラインにSAST/DAST/SCA/シークレットスキャン/IaCスキャンを組み込み、コミット時点から全チームの変更に自動適用しているか
- **なぜ重要か**: パイプラインが唯一の変更経路になれば組織全体への施策展開が可能になり、人手チェックの漏れを防げる。リリース後修正よりコミット前修正の方が低コスト(OSSの脆弱性修正は平均271日かかるとの調査もある)。
- **出典**: Cloud Native Security Cookbook 第1章1.5・第7章7.7-7.9; Defensive Security Handbook 2nd Ed 19章; Identity Security for Software Development 2・6・9章; Security Architecture for Hybrid Cloud 第10章; Software Supply Chain Security 第4章(p.59-60); Web Application Security 2nd Ed 第26章

### 2. npm/PyPI/Docker Hub等の公開パッケージレジストリを直接利用せず、プライベートアーティファクトリポジトリ経由のプロキシ・イベント駆動の脆弱性スキャンに限定し、タイポスクワッティング・dependency confusion・ブランドジャッキング対策を行っているか
- **なぜ重要か**: 既知脆弱性スキャン(SCA)だけでは防げない「未知の悪意あるパッケージの混入」という別の攻撃ベクトルであり、OWASP CI/CD Top10でも典型的な攻撃手法とされる。
- **出典**: Identity Security for Software Development 9章; Software Supply Chain Security 第5章(p.84, Control SCBD-09); Web Application Security 2nd Ed 第17章

### 3. IDE/エディタのプラグインでコーディング中(コミット前)にリアルタイムのセキュリティフィードバックを行っているか
- **なぜ重要か**: シフトレフトを最も手前まで徹底する手段だが、IDE自体も攻撃対象になり得る(Octopus Scanner攻撃の実例)。
- **出典**: Software Supply Chain Security 第5章(p.79)

### 4. IaC(Terraform/CloudFormation等)のコードをアプリケーションコードと同様にバージョン管理し、コミット権限を制限した上でピアレビュー+スキャンを必須化しているか
- **なぜ重要か**: 手動変更より監査可能性が高く、IaC自体への不正アクセス防止が最優先事項の一つとされている。
- **出典**: Defensive Security Handbook 2nd Ed 14章・19章; Security as Code 第3章(p.21-22)

### 5. ビルド成果物にSBOM(SPDX/CycloneDX)とSLSA provenanceを付与し、ビルド入出力の暗号学的検証・HSMで保護した署名鍵によるコード署名(Cosign/sigstore等、重要リリースは複数者承認の署名式)・再現可能ビルド(Reproducible Builds)を実施しているか
- **なぜ重要か**: SolarWinds事件のようなビルドプロセス自体への侵入は検知まで約14ヶ月かかった例もあり、出自(provenance)の証明と改ざん検知の仕組みが必要。
- **出典**: Identity Security for Software Development 9章; Software Supply Chain Security 第5章(p.81)・第8章(p.129-144); Security Architecture for Hybrid Cloud 第10章

### 6. 本番環境への変更はIaC/CI-CDパイプライン(サービスアカウント/マネージドID経由)からのみ行われるようにし、ユーザーアカウントによる手作業での直接操作を禁止・検知しているか
- **なぜ重要か**: 本番変更は自動化経路からのみ行われるべきで、手作業デプロイは設定ミス・属人化・構成ドリフトと不可視な侵害経路を生む。
- **出典**: Cloud Native Security Cookbook 第11章11.1-11.3(p.490); Security Architecture for Hybrid Cloud 第9・10章; Software Supply Chain Security 第6章(p.106, Control CLD-06)

### 7. Terraform等のIaC stateファイルのストレージでバージョニングを有効化し、アクセスを厳格に制御しているか
- **なぜ重要か**: stateには機密情報が含まれ、破損・消失時の復旧が極めて困難になる。
- **出典**: Cloud Native Security Cookbook 第6章6.7-6.9(p.313,320,327)

### 8. cfn-nag/Parliament等のIaC静的解析(リンター)でテンプレートをスキャンし、WARN/FAILを可視化しているか
- **なぜ重要か**: 実際のスキャン出力ログ(W2, W9, F1000等)が示すように、過剰許可やegress未定義といった設定ミスを本番到達前に検出できる。
- **出典**: Security as Code 第3章(p.30-33)・第5章(p.64-66)

### 9. シークレット管理・アクセス制御・ネットワークポリシー等をPolicy as Code(Rego/Cedar/Conjur等)で記述し、通常のソフトウェア開発と同様にバージョン管理・テスト・レビューの対象としているか
- **なぜ重要か**: 手動管理に比べ一貫性があり、ポリシー変更の履歴追跡・レビューをコードと同じ基準で行うことで一貫性を確保できる。
- **出典**: Identity Security for Software Development 8章; Zero Trust Networks 2nd Ed Ch4(p.59), Ch9(p.215-216)

### 10. ポリシー変更を「ログのみ記録(monitor)→段階的強制→全面強制」の順で安全にロールアウトしているか
- **なぜ重要か**: 急な全面強制は正規の通信・業務を誤って遮断するリスクがあるため、段階的ロールアウトで誤検知を洗い出す必要がある(PagerDutyのIPsec none→use→required移行が好例)。
- **出典**: Zero Trust Networks 2nd Ed Ch9(p.215-216, 236-237)

### 11. Chef/Puppet/Ansible等の構成管理ツールを用いて本番環境の構成ドリフトを継続的に検知・防止しているか
- **なぜ重要か**: 構成管理システムはゼロトラスト制御プレーンの実装基盤にもなり、手動変更によるドリフトを防ぐ。
- **出典**: Zero Trust Networks 2nd Ed Ch5(p.97-98), Ch9(p.211-212)

### 12. コンプライアンスポリシー(Conformance Packs等)を組織の上位階層(親アカウント)にデプロイし、下位チーム(子アカウント)が無効化・改変できない構成にしているか
- **なぜ重要か**: ガバナンスの強制力を担保するため。
- **出典**: Security as Code 第3章(p.50)

### 13. パイプラインに決定ゲート(Decision Gate)を設け、セキュリティチェックでエラーが検出された場合は自動的にデプロイを中止しているか
- **なぜ重要か**: 人の承認待ちにせず機械的に止める設計のほうが確実に機能する。
- **出典**: Security as Code 第1章(p.6, 9)

### 14. Preventive(予防)・Detective(検知)・Corrective(是正)の3種類のコントロールをすべて実装し、まず予防を優先しているか
- **なぜ重要か**: 検知のみでは「公開されていた時間」がリスクを増大させ、防止への投資の方が修復依存よりコンプライアンス債務を増やさない。
- **出典**: Security as Code 第3章(p.24-25); Cloud Native Security Cookbook 第7章扉/7.12(p.341,405)

### 15. 緊急時のセキュリティチェックのバイパス機能を安易に用意せず、使う場合は必ず記録・事後是正するプロセスがあるか
- **なぜ重要か**: 「今回だけ」の例外が常態化するリスクがあるため。
- **出典**: Security as Code 第1章(p.7)

### 16. セキュリティチェックの自動化が「冪等性」「ベースライン(大多数のリソースに適用可能な汎用性)」「推奨される修正方法の提示」の3条件を満たしているか
- **なぜ重要か**: ノイズが多く修正方法を示さない自動化は形骸化する。
- **出典**: Security as Code 第1章(p.9-10)

### 17. 発見されたIaC上のセキュリティ不備の修正責任を、作った開発チーム自身に持たせているか(セキュリティチームは支援役にとどめているか)
- **なぜ重要か**: セキュリティチームへの修正業務集中はスケールしない。
- **出典**: Security as Code 第7章(p.93, 97)

### 18. 脅威モデリング(STRIDE等)を設計時の一度きりで終わらせず、継続的に更新する"living artifact"として運用し、脅威とコントロールのトレーサビリティ(マトリクス)を維持しているか
- **なぜ重要か**: 脅威モデルは完成しないことを前提にすべきであり、業界標準(CSA CCM等)だけではカバーできない自社固有のリスクを補完する必要がある。
- **出典**: Security as Code(p.24, 93-95); Security Architecture for Hybrid Cloud 第6章

### 19. 開発環境自体をInfrastructure as Codeで事前定義し、作成・削除を自動化しているか
- **なぜ重要か**: シャドーIT化した開発/ラボ環境は標準ポリシーの管理外になりやすい。
- **出典**: Software Supply Chain Security 第3章(p.40-41, Control IS-01)

### 20. セキュリティ成熟度モデル(BSIMM, OWASP SAMM等)で組織のAppSec成熟度を定期評価し、施策の優先順位付けに使っているか
- **なぜ重要か**: 成熟度に応じた優先順位付けが可能になる。
- **出典**: Software Supply Chain Security 第4章(p.69)

### 21. 生成AIコーディング支援ツールの利用について、生成コードに対するセキュリティレビューを必須化する方針を定めているか
- **なぜ重要か**: AI支援利用者はより安全性の低いコードを書きながら「安全だと過信する」傾向が確認され、生成コードの約40%に脆弱性が含まれたとの調査結果がある(Cornell大学・NYU)。
- **出典**: Security Architecture for Hybrid Cloud 第10章; Software Supply Chain Security 第5章(p.77-78)・第8章(p.144)

### 22. セキュリティ施策の効果を定量指標(通知/修正リードタイム、コンプライアンス率、変更拒否率、防止侵害数等)で継続的に測定しているか
- **なぜ重要か**: 「測定されるものは管理される」―定性的な安心感だけでは改善サイクルが回らない。
- **出典**: Cloud Native Security Cookbook 第1章1.6(p.12-14)

### 23. 修正済み脆弱性に対するリグレッションテストをCI上で必須化しているか
- **なぜ重要か**: 大企業の実例で脆弱性の約25%が過去に閉じたバグの再発だったと報告されている。
- **出典**: Web Application Security 2nd Ed 第20・26章
