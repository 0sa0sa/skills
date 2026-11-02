# サプライチェーンセキュリティ

SBOM、依存関係、ビルド来歴、CI/CD改ざん対策、署名検証に関わる設計・実装を見る。

## チェックリスト

### 1. ソフトウェアの本番リリースごとに、SBOM(ソフトウェア部品表)をビルドプロセス中(prebuild/build/postbuild)に生成し、継続的に維持しているか
- **なぜ重要か**: ビルド後の静的解析だけでは独自・商用ライブラリの構成要素を認識できずSBOMに欠落が生じる。Log4Shellのように利用箇所の特定が困難な脆弱性や、SolarWinds・3CX/X_TRADERのような二重サプライチェーン攻撃の実例を踏まえても、SBOMの生成・維持自体が前提として不可欠である。
- **出典**: Software Supply Chain Security 第8章(p.132, Control ST-01); Security Architecture for Hybrid Cloud 第10章; Zero Trust Networks, 2nd Edition 第7章(p.138-139)

### 2. SBOMをCycloneDX/SPDX/SWID等の業界標準フォーマットで、NTIA最小要素(作成者・タイムスタンプ・サプライヤー・コンポーネント・バージョン・依存関係等)を満たして生成しているか
- **なぜ重要か**: 米国政府提出要件等の最低ラインを満たさないSBOMは不完全で、実運用で使えない。
- **出典**: Software Supply Chain Security 第8章(p.133-135, Table 8-1/8-2)

### 3. SBOMとVEX(Vulnerability Exploitability eXchange)/VDRを組み合わせ、脆弱性が実際に悪用可能かを評価しているか
- **なぜ重要か**: SBOM単体では「リスクの所在」しか分からず、「悪用可能かどうか」の判断ができないため。
- **出典**: Software Supply Chain Security 第8章(p.129-130, 137-139)

### 4. SBOMの記載内容を実際に稼働している資産のバージョン(IT資産台帳)と突合し、バックポート済みパッチの反映状況も確認しているか
- **なぜ重要か**: 稼働中バージョンとSBOM記載バージョンの不一致やバックポートの見落としは「偽の安心感」を生み、脆弱性有無を誤判定させる。
- **出典**: Software Supply Chain Security 第8章(p.136)

### 5. 依存関係(直接依存だけでなく推移的/サブ依存を含む)を自動化ツール(Dependabot、Snyk、npm ls、NVD照合等)で継続的に脆弱性スキャンしているか
- **なぜ重要か**: 大規模アプリケーションは数万のサブ依存を持ち手動レビューでは把握不可能であり、サブ依存1つの侵害が広範囲に影響した実例(event-stream等)がある。
- **出典**: Identity Security for Software Development 第9章; Web Application Security 第17章・第35章

### 6. 依存ライブラリに既知の脆弱性が見つかった際の「通知〜修正」のリードタイムをメトリクスとして測定し、改善サイクルに反映しているか
- **なぜ重要か**: 外部ライブラリ依存が無数にある以上、通知・修正の速度こそが被害範囲を左右する。
- **出典**: Cloud Native Security Cookbook 第1章(p.12-13)

### 7. lockfile(package-lock.json等)やGit SHA固定でパッケージの意図しないバージョン更新(caretバージョン指定の自動更新等)を防いでいるか
- **なぜ重要か**: パッケージマネージャのcaretバージョン指定は意図しないパッチ更新を許し、サプライチェーン上の「バージョン再利用」リスクを生む。
- **出典**: Web Application Security, 2nd Edition 第35章

### 8. 外部パッケージレジストリ(npm/PyPI/Docker Hub等)を直接利用せず内部ホスティングのプライベートアーティファクトリポジトリ経由でプロキシ・脆弱性スキャンを行い、タイポスクワッティングやdependency confusion対策をしているか
- **なぜ重要か**: 2022年にPyPIで偽PyTorchパッケージが本物を約1週間置き換えた実例があり、公開レジストリは少数のボランティア運営で侵害対応が遅れがちなため。
- **出典**: Identity Security for Software Development 第9章; Software Supply Chain Security 第5章(p.84, Control SCBD-09)

### 9. 過去の実際のOSSサプライチェーン事件(left-padの削除、eslint-scopeの資格情報侵害、event-stream→flatmap-streamによるビットコイン窃取等)を踏まえ、メンテナ権限侵害や悪意あるパッケージ公開のリスクを自組織の評価に反映しているか
- **なぜ重要か**: サブ依存1つの侵害が広範囲に影響した実例が複数あり、「OSSは商用コードほど厳格に審査されない」という前提でリスクを見積もる必要がある。
- **出典**: Web Application Security, 2nd Edition 第17章

### 10. サポート終了(EOL)となったOS・ミドルウェア・ライブラリを特定し、ネットワークから隔離または更新しているか
- **なぜ重要か**: パッチが提供されず既知脆弱性が残存し続け、実際に大規模サーバー群が旧式OSで稼働していた事例(Twitter)がある。
- **出典**: Defensive Security Handbook, 2nd Edition 第10-11章(p.99-100, 110-112); Inside Cyber Warfare, 3rd Edition 第1章(p.12)

### 11. ハードウェア・ファームウェアのサプライチェーンについて、HBOM(ハードウェア部品表)を活用し、チップ製造・PCB組立・最終組立・梱包の各工程でチェーンオブカストディを追跡しているか。またファームウェアへのデジタル署名、TPM等のハードウェアルートオブトラスト、セキュアブートを導入しているか
- **なぜ重要か**: 製造工程が多段にわたるほど改ざんの機会が増え、2020-2022年の半導体不足下では偽造・クローンチップの流通も増加した。
- **出典**: Software Supply Chain Security 第10章(p.172-176, Control MDS-03/05)

### 12. ビルド・CI/CDパイプラインの実行環境をエフェメラル(使い捨て)にし、チェックサム(SHA256等)で再現可能な完全性を検証しているか
- **なぜ重要か**: 静的な(使い回しの)ビルド環境は持続的な改ざんの標的になりやすい。
- **出典**: Software Supply Chain Security 第5章(p.86, Control SCBD-10)

### 13. ソースコード・ビルド・デプロイ各段階の出所(provenance)情報(リポジトリアクセスログ、CI/CDログ、生成AI使用データ等)を記録しているか
- **なぜ重要か**: 来歴不明なコードは改ざんやライセンス違反を隠しうる。
- **出典**: Software Supply Chain Security 第8章(p.143-144, Control ST-05)

### 14. SLSAフレームワークを用いて、自組織のソース完全性・ビルド完全性が現在どのレベル(L0〜L3)にあるかを特定し、未達成のコントロールへの対応計画を文書化しているか
- **なぜ重要か**: SolarWinds攻撃を受けてGoogleが開発した体系的な対策であり、達成度を測る具体的な指標になる。
- **出典**: Software Supply Chain Security 第5章(p.81); Cloud Native Security Cookbook 第10章(p.484); Identity Security for Software Development 第9章

### 15. SCITT・in-toto attestationのような来歴(provenance)検証フレームワークの導入を評価しているか
- **なぜ重要か**: ビルド・デプロイの各ステップでの改ざんをエンドユーザー側が検証できるようになる。
- **出典**: Software Supply Chain Security 第8章(p.142-143)

### 16. 再現可能ビルド(Reproducible Builds)を採用し、第三者が同一のバイナリを独立して再生成・検証できるようにしているか
- **なぜ重要か**: ビルドプロセス自体の改ざんを外部から検証可能にするため。
- **出典**: Zero Trust Networks, 2nd Edition 第7章(p.146)

### 17. CI/CDパイプライン内のシークレット(APIキー、クラウド資格情報等)が、グローバルスコープで全ジョブ・全ユーザーに公開される設定になっていないか
- **なぜ重要か**: 低権限ユーザーが高権限シークレットに到達できてしまう構造的リスクがあり、JenkinsのwithCredentialsではこの問題が具体的に指摘されている。
- **出典**: Identity Security for Software Development 第9章

### 18. mainブランチへの直接pushを禁止し、作成者と異なるレビュアーによるPR承認、および署名付きコミット(GPG)を強制しているか
- **なぜ重要か**: XZ Utilsバックドア事件は信頼されたメンテナ自身がコードを操作した例であり、レビュー・署名の強制がなければ防げなかった。
- **出典**: Identity Security for Software Development 第9章; Zero Trust Networks, 2nd Edition 第7章(p.141-143)

### 19. 自組織がビルドする成果物に対し、SBOM・SLSA provenance・デジタル署名(Cosign/sigstore等)を付与し、下流の利用者が改ざんを検知できるようにしているか
- **なぜ重要か**: SolarWinds事件のようなビルドプロセス自体への侵入を、下流の利用者側が検知・防止できる仕組みが必要。
- **出典**: Identity Security for Software Development 第9章

### 20. 配布するコード・ドライバ・スクリプト・アプリケーションについて、信頼できるCA(認証局)の鍵による署名とハッシュによる完全性検証を組み合わせて二重に検証し、かつTLSで保護された配布網を使用しているか(非TLSミラーへのダウングレード攻撃にも注意し、署名済みであっても安全とは限らない点にも留意しているか)
- **なぜ重要か**: ハッシュまたは署名の片方だけ、あるいは脆弱な配布網への迂回を許す構成では完全性を防げない。NVIDIA秘密鍵盗難事件やCodecov事件(ハッシュ不一致が顧客に気づかれるまで2か月間検出されなかった)、正規署名済みパッケージへのマルウェア混入事例(Comm100, 2022年)が示すように、署名の有無だけでは完全性を保証できない。
- **出典**: Software Supply Chain Security 第5章(p.87-88, Control SCBD-11/12); Zero Trust Networks, 2nd Edition 第7章(p.148-151)

### 21. コード署名用の秘密鍵をHSM(ハードウェアセキュリティモジュール)で保護し、重要リリースには複数者承認による署名式(code signing ceremony)を実施しているか
- **なぜ重要か**: SolarWinds SUNBURST事件はコードサイニング資格情報の管理不備が一因であった。
- **出典**: Zero Trust Networks, 2nd Edition 第7章(p.151-152); Identity Security for Software Development 第2章

### 22. サプライヤー評価において、IT部門の評価だけでなく開発・ビルド・本番相当環境へのペネトレーションテスト結果を要求し、SOC2/ISO27001証明書のみに依存していないか
- **なぜ重要か**: これらの証明書はセキュア開発プラクティス自体を直接評価するものではなく、多くの組織は開発環境をペンテスト範囲に含めていない。
- **出典**: Software Supply Chain Security 第9章(p.153-158, Control SP-03/SP-05/SP-10)

### 23. 重要なサプライヤーとの契約にSDL要件・脆弱性/パッチ対応のSLA(重大=30日等)・契約終了時のコードエスクローを明記し、定期レビューと監査権(right to audit)を保持しているか
- **なぜ重要か**: 契約終了時対応はサプライヤー契約管理で最も見落とされがちな項目であり、サプライヤーのセキュリティ姿勢は時間とともに悪化しうる。
- **出典**: Software Supply Chain Security 第9章(p.160-164, Control SP-13/SP-14)

### 24. CISAのKEV(実際に悪用された既知脆弱性)カタログ等と照合し、全CVEを均等に追うのではなく実害リスクの高い脆弱性へパッチ適用の優先順位を絞っているか
- **なぜ重要か**: 全CVEを追うアプローチは非現実的であり、実際に悪用されている脆弱性に絞り込む仕組みが必要。
- **出典**: Inside Cyber Warfare, 3rd Edition 第1章(p.13-14), 第3章(p.37-38)

### 25. 生成AIが書いたコードについて、ライセンス・著作権・学習データの出所を法務・セキュリティでレビューしているか
- **なぜ重要か**: 生成コードの約40%に脆弱性が含まれるとの研究があり、GitHub Copilot訴訟のようにAI生成コードの権利的位置づけも未解決である。
- **出典**: Software Supply Chain Security 第5章(p.77-78), 第8章(p.144)
