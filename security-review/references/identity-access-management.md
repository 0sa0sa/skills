# IDとアクセス管理

認証・認可、シークレット管理、最小権限、MFA、特権アクセス管理、機械IDに関わる設計・実装を見る。

## チェックリスト

### 1. 全リモートアクセス・VPN・高権限アカウントにMFAを必須化し、特にフィッシング耐性のある方式(FIDO2/U2F)をSMS・TOTP・push通知より優先しているか
- **なぜ重要か**: push通知型はMFA疲労(フラッディング)攻撃に弱く、SMSは中間者攻撃に弱い。2014年JPMorgan侵害(8300万件漏洩)はMFA未適用サーバーが突破口であり、2022年Uber侵害もMFA疲労攻撃が起点だった。
- **出典**: Defensive Security Handbook 2nd Ed 15章(p.193); Zero Trust Networks 2nd Ed Ch6(p.121,133-134); Identity Security for Software Development 1章・3章

### 2. MFAを導入する際、固定要素(パスワード+OTP等)だけでなく、異常なログイン位置・デバイス・時間帯を検知して追加認証を要求するリスクベース/適応型認証(adaptive authentication)を導入しているか
- **なぜ重要か**: push通知疲労攻撃のような単純なMFA回避手口に対し、ログインのコンテキスト(位置・デバイス・時間帯)の異常検知を組み合わせることで追加の防御層になる。
- **出典**: Identity Security for Software Development 1章・3章

### 3. パスワードポリシーで最低10文字以上・大小英数記号混在を強制し、SHA-512+ソルトやbcrypt/PBKDF2等の低速鍵導出関数でハッシュ保存しているか(MD5/SHA1/DES等は不可)
- **なぜ重要か**: 8文字小文字のみは2分未満で解読可能、10文字混在は約600年必要という試算がある。ソルトなしハッシュ・高速アルゴリズムはレインボーテーブル/ブルートフォースに脆弱。
- **出典**: Defensive Security Handbook 2nd Ed 15章(p.176,178-180); Web Application Security 2nd Ed 21章; Zero Trust Networks 2nd Ed Ch6(p.120-121)

### 4. Basic/Digest/OAuth等の認証方式を選定する際、強度とリスクのトレードオフを文書化しているか
- **なぜ重要か**: HTTP Basic認証のような単純な方式は傍受・XSSで容易に漏洩するため、方式選定の根拠を残すことで後からの見直し・妥当性確認が可能になる。
- **出典**: Web Application Security 2nd Ed 第3章・第5章

### 5. AuthN(本人確認)とAuthZ(権限付与)を設計上明確に分離し、別の実装要件として扱っているか
- **なぜ重要か**: 両者を混同すると「認証できないものを認可する」といった誤りが生じやすい。
- **出典**: Identity Security for Software Development 2章・3章

### 6. RBAC(役割ベース)・ABAC(属性ベース)・PBAC(ポリシーベース)などのアクセス制御モデルを採用しているか
- **なぜ重要か**: OWASP Top10「A01:2021 Broken Access Control」の筆頭対策であり、役割・属性・ポリシーに基づく体系的なアクセス制御は個別ハードコードされた権限チェックより保守性・監査性が高い。
- **出典**: Identity Security for Software Development 4章; Cloud Native Security Cookbook 第9章9.2(p.467)

### 7. ABACを導入する場合、設定ミスのリスク(学習曲線の高さ)を認識し、まずRBACで運用を確立したうえで段階的にABACへ移行しているか
- **なぜ重要か**: ABACは強力だが学習曲線が高く、属性の組み合わせが複雑になるほど設定ミスが発生しやすい高度技術である。
- **出典**: Identity Security for Software Development 4章

### 8. 人的アカウント・マシンアカウント双方に最小権限の原則を適用し、付与済み権限の棚卸し(エンタイトルメントレビュー)を定期的に実施して、使われていない・不要になった権限を縮小・剥奪しているか(privilege creepの継続監査)
- **なぜ重要か**: 権限は業務上の必要に応じて拡大する一方で自然には縮小されないため、時間とともに過剰権限化(privilege creep)するのが最も多い問題である。
- **出典**: Cloud Native Security Cookbook 第1章1.7(p.14); Security as Code 第5章(p.58-59); Security Architecture for Hybrid Cloud 第3章「Least Privilege」; Defensive Security Handbook 2nd Ed 14章(p.156)

### 9. 退職者・異動者・不要アカウント・休眠アカウントへのアクセスを即時かつ自動でdeprovisioning(削除)しているか。また職務分離(SoD)を組み込み、一人の人物が申請から承認までを単独で完結できない設計になっているか
- **なぜ重要か**: deprovisioning遅延は休眠アカウントという攻撃対象を放置することになり、SoDはIGAの中核機能でコンプライアンス報告の根拠にもなる。
- **出典**: Identity Security for Software Development 4章

### 10. 恒久的な特権付与ではなく、必要なときだけ権限を有効化するJust-In-Time・時間制限・承認制の特権アクセス管理(PAM、Azure PIM等)を導入しているか。クラウドでは常時は無権限状態を保つゼロ・スタンディング・プリビレッジ(ZSP)とTEA(Time, Entitlements, Approvals)方式を検討しているか
- **なぜ重要か**: 恒久的特権は漏洩時の影響範囲・期間を増大させ、クラウドの動的性質上「常にオンの権限」は管理困難で攻撃対象を広げる。
- **出典**: Cloud Native Security Cookbook 第8章8.3(p.421); Identity Security for Software Development 6章; Security Architecture for Hybrid Cloud 第8章「Identity and Access Management」

### 11. Domain Admins等の特権グループのメンバーを必要最小限に絞り、特権アカウント(sysadmin等)を集中させず攻撃の主要標的になり得ることを前提に監視しているか
- **なぜ重要か**: 「とりあえずDomain Admin」は監査不能な過剰権限の温床になり、LAPSUS$によるSamsung/NVIDIA/Okta等への侵入は特権アカウント集中が一因だった。
- **出典**: Defensive Security Handbook 2nd Ed 10章(p.106); Zero Trust Networks 2nd Ed Ch2(p.35)

### 12. ローカル管理者パスワードをLAPS等でランダム化・自動ローテーションしているか
- **なぜ重要か**: パスワード使い回しはラテラルムーブメントを容易にする。
- **出典**: Defensive Security Handbook 2nd Ed 10章(p.106)

### 13. マルチアカウント/マルチクラウドで長期間有効なIAMユーザー・長期固定の資格情報(アクセスキー等)の発行を避け、ロール引き受け(AssumeRole等)による短命な一時的認証情報を基本方針としているか
- **なぜ重要か**: 長期資格情報は漏洩時にその資格情報が使える期間が長く、被害が長期化・拡大する(blast radius)。
- **出典**: Cloud Native Security Cookbook 第2章2.8(p.57-58); Identity Security for Software Development 1章・6章(blast radius)

### 14. AWSのPermission Boundary/SCP、Azure PolicyなどでIAMロール・プリンシパルが自分より高い権限へ自己昇格できないよう制限しているか(Azureの「User Access Administrator」のデフォルト自己昇格仕様等、既知の盲点を認識しているか)
- **なぜ重要か**: 境界がなければ権限昇格を防げず、Azureでは既定動作のままだと管理者ロール=実質Owner権限になってしまう。
- **出典**: Cloud Native Security Cookbook 第8章8.2-8.3(p.415,419-420); Security as Code 第5章(p.62-63)

### 15. IAMポリシーにリソース指定のワイルドカード(`"Resource": "*"`等)がないか、タグベースのアクセス制御も含めて自動スキャン(IAM Access Analyzer等)しているか
- **なぜ重要か**: ワイルドカード許可は過剰権限の温床であり、タグベース制御はスケールする権限管理の基盤になる。
- **出典**: Security as Code 第5章(p.60-61,65-66)

### 16. 人間のIDだけでなく機械ID(サービスアカウント、CI/CDパイプライン、AIエージェント、IoTデバイス等)のライフサイクル管理(発行・ローテーション・失効)を自動化しているか。人的アイデンティティを本番環境からできる限り遠ざけ、予測可能なマシンアイデンティティ経由の操作に置き換えているか
- **なぜ重要か**: CyberArk 2025年調査では機械IDは人間の82倍存在し急増中であり、人的ミスの排除にもつながる。
- **出典**: Identity Security for Software Development 1章; Security as Code 第5章(p.61)

### 17. SPIFFE/SPIREのようなワークロードアイデンティティ基盤を用いてマイクロサービス間/サービスメッシュの認証を行い、シークレット配布への依存を減らしているか(自動発行・ローテーション)
- **なぜ重要か**: SPIFFE IDは暗号材料に基づく検証可能なワークロードIDで、秘密情報の受け渡し自体を不要化できる。
- **出典**: Identity Security for Software Development 7章; Zero Trust Networks 2nd Ed Ch6(p.126-127)

### 18. 本番コード・スクリプト・設定ファイル・README・環境変数・ビルドマニフェスト・ログにハードコードされた認証情報が存在しないか確認しているか
- **なぜ重要か**: 2022年Uber侵害はPowerShellスクリプト内のハードコード管理者資格情報が侵害の起点だった。
- **出典**: Identity Security for Software Development 1章・2章・3章

### 19. シークレット(APIキー、暗号鍵、クラウド資格情報等)をVault/Key Vault/Secrets Manager等で一元管理し、CI/CDパイプラインやIaCコード・コンテナイメージに対してシークレットスキャンを行っているか
- **なぜ重要か**: ハードコードされた認証情報は代表的な悪習慣の筆頭であり、SolarWinds侵害ではMimecastの証明書窃取によりMicrosoft 365通信が傍受された。
- **出典**: Defensive Security Handbook 2nd Ed 14章(p.155,162); Software Supply Chain Security 第7章(p.118); Security Architecture for Hybrid Cloud 第10章

### 20. シークレット管理ツールが組織内に分散して「セキュリティの島」を形成していないか確認し、統合を進めているか
- **なぜ重要か**: ツールの分散はシャドーIT化を招き、監査・管理コストを増大させる。
- **出典**: Identity Security for Software Development 5章

### 21. シークレット管理を集中化する場合、そのマスターキー/ルート認証情報が単一障害点となる「secret zero」のリスクを評価し、対策(多重防御、break-glassプロセス等)を講じているか
- **なぜ重要か**: 集中管理はツールの分散という問題を解決する一方、マスターキー1つが破られると全滅するという新たなトレードオフを抱える。
- **出典**: Identity Security for Software Development 5章

### 22. CI/CDパイプライン内のシークレットがグローバルスコープ(JenkinsのwithCredentials等)で全ジョブ・全ユーザーに公開されていないか確認しているか
- **なぜ重要か**: 低権限ユーザーがGoogle Cloud資格情報等の高権限シークレットに到達できてしまう構造的リスクがある。
- **出典**: Identity Security for Software Development 9章

### 23. JWTを使う場合、有効期限(exp)を必ず設定し、ペイロードを軽量化し、常にAuthZサーバ側で再検証しているか。OAuth 2.0ではインクリメンタルな権限要求を行い、本番環境で開発時の過剰スコープを削除しているか
- **なぜ重要か**: JWTは自己完結型トークンのため検証をアプリ判断だけに頼ると改ざんトークンを受理するリスクがあり、使われない機能のためのアクセス要求は不要な攻撃対象領域を増やす。
- **出典**: Identity Security for Software Development 3章

### 24. Cookieに`HttpOnly`/`Secure`/`SameSite=Strict`/`Path`属性を適切に設定しているか
- **なぜ重要か**: XSS発生時のセッショントークン窃取や、CSRFによる意図しないCookie送信を防ぐ。
- **出典**: Web Application Security 2nd Ed 第22章

### 25. 権限変更(解雇・降格等)後、既存の認証トークン/セッションが即座に無効化される継続的認可・継続的認証の仕組みがあるか(セッション中の振る舞い・デバイス・コンテキストの再評価を含む)
- **なぜ重要か**: 48時間有効なトークンが解雇後も有効なままだと不正操作が可能であり、「2日前に開始したセッションが本人か分からない」という問題が生じる。
- **出典**: Web Application Security 2nd Ed 21章; Security Architecture for Hybrid Cloud 第2章「Continuous authentication」

### 26. SSOトークンをアプリ側キャッシュに任せず、コントロールプレーンで頻繁に再検証しているか
- **なぜ重要か**: アプリ独自トークンへの委任は失効を困難にする。
- **出典**: Zero Trust Networks 2nd Ed Ch6(p.118-119,124-125)

### 27. EDR相当の脅威検知層をID(アイデンティティ)層にも導入(ITDR)し、人的・サービス双方の全アカウントの異常行動を監視しているか。特に特権アカウントについてはSOC経由で継続的に監視しているか
- **なぜ重要か**: ハイブリッドクラウドではID数が爆発的に増えるため、ID層自体への脅威検知が必要であり、アカウント乗っ取りの監視はSolarWinds型攻撃のような侵入の起点を早期に検知する手段になる。
- **出典**: Security Architecture for Hybrid Cloud 第8章「Identity and Access Management」; Software Supply Chain Security 第3章(p.44)

### 28. 認可チェックを一元化されたクラスで実施し、API/エンドポイントごとに個別実装していないか。オブジェクト参照(IDOR)には推測困難なID/ファイル名とアクセス権チェックを用いているか
- **なぜ重要か**: 認可の個別実装は人的ミスによる欠落が発生しやすく、連番IDは直接アクセスで他人のリソースに到達できてしまう。
- **出典**: Web Application Security 2nd Ed 第3章・第15章・第33章

### 29. 署名鍵・暗号鍵をHSM(ハードウェアセキュリティモジュール)で管理し、鍵のローテーションが実際に機能しているか確認しているか。コードサイニング鍵は暗号化・安全な場所に保管され、アクセス権限者が明確に定義されているか
- **なぜ重要か**: Microsoft Storm-0558侵害(2023)では2016年製の署名鍵が盗まれた経緯が未解明のまま放置されていた。SolarWinds SUNBURST事件もコードサイニング資格情報の管理不備が一因だった。
- **出典**: Inside Cyber Warfare 3rd Ed 第3章(p.34-36); Identity Security for Software Development 2章

### 30. データ保管時の暗号化でBYOK(Bring Your Own Key)やHSMによる鍵管理を検討し、クラウドプロバイダーの特権ユーザーからもデータを保護できているか
- **なぜ重要か**: プロバイダー管理鍵だけではプロバイダー従業員からの保護にはならない。
- **出典**: Security Architecture for Hybrid Cloud 第8章

### 31. コードリポジトリ・ビルド環境へのアクセスをIAM/SSOに接続し、MFAを必須化した上で、最小権限・need-to-know原則に基づきread-only/privileged/adminの権限レベルを分離しているか
- **なぜ重要か**: 内製の個別アクセス管理は剥奪漏れによる漏洩リスクが高く、権限レベルが分離されていないとインターンが初日に本番データベースを削除できるような事態を招く。
- **出典**: Software Supply Chain Security 第3章(p.43-44, Control IS-03)

### 32. 複数CSP(AWS/GCP/Azure)やIdP切替時に、IAMモデルの違い(JSONポリシー vs RBAC、権限とリソースの結合/分離)を理解し、マッピングを文書化しているか
- **なぜ重要か**: 各CSPのIAM実装は構造が異なり、移行時の見落としが過剰/不足権限を生む。
- **出典**: Identity Security for Software Development 6章

### 33. LLM APIのリクエストにJWT等によるトークン認証を実装し、有効期限とブラックリスト(失効管理)を設けているか。ユーザー種別(basic/premium/admin)ごとにアクションレベルの権限(generate_text/fine_tune/deploy等)を分離しているか
- **なぜ重要か**: トークン漏洩・リプレイ攻撃対策であり、最小権限原則の欠如は過剰な機能露出につながる。
- **出典**: Privacy and Security for Large Language Models 第5章(Authentication and Authorization, PermissionsManager)

### 34. 高感度操作には単独の人間を信頼しない仕組み(グループ認証/Shamir's Secret Sharing等)を適用し、アカウントリカバリ手段(秘密の質問等)の脆弱性を評価し、アイデンティティのブートストラップ(対面確認等)とオンボーディング/オフボーディングを自動連携しているか
- **なぜ重要か**: 単独アクセスは強要(coercion)や乗っ取りに弱く、著名人のAmazonアカウント乗っ取り事例のようにリカバリ手段そのものが攻撃経路になりうる。
- **出典**: Zero Trust Networks 2nd Ed Ch6(p.113-116,128-129)
