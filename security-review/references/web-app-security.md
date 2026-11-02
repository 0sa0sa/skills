# Webアプリケーションセキュリティ

OWASP系脆弱性(インジェクション、XSS、CSRF、認証不備、ビジネスロジック脆弱性)に関わる設計・実装を見る。

## チェックリスト

### 1. SQL/コマンド実行をプリペアドステートメント(パラメータ化クエリ)とallowlistで構築し、文字列連結やブロックリスト方式に依存していないか
- **なぜ重要か**: ブロックリストはエンコーディング変換で回避され、文字列連結はクエリ構造自体を書き換えられる。SQLiはxp_cmdshell再有効化等によりOSコマンド実行まで到達しうる。
- **出典**: Web Application Security 2nd Edition 第13章・第31章 / Identity Security for Software Development 第1・2・4章 / Defensive Security Handbook 2nd Edition 第13章 p.138-139

### 2. 入力検証をクライアント側だけでなく必ずサーバー側でも実施し、型・長さ・形式(許可文字)をallowlistベースで検証しているか
- **なぜ重要か**: クライアント側検証のみは「クライアントからの通信は信頼できる」という誤った前提に基づき、ブラウザ脆弱性やMITM攻撃で容易に回避される。
- **出典**: Identity Security for Software Development 第2章

### 3. XMLパーサーで外部エンティティ解析を明示的に無効化しているか(SVG/PDF/RTF等のXML系形式も含めて確認したか)
- **なぜ重要か**: 設定漏れがXXE→ファイル漏洩→アカウント乗っ取りに直結する。
- **出典**: Web Application Security 2nd Edition 第12章・第30章

### 4. 複雑な正規表現に対しReDoS(evil regex)耐性をレビュー・静的解析しているか
- **なぜ重要か**: 貪欲マッチ型の正規表現は入力長に対し指数的に遅くなり、サービス拒否(DoS)を引き起こす。
- **出典**: Web Application Security 2nd Edition 第14章・第32章

### 5. オブジェクトのmerge/assign処理で`__proto__`/`constructor.prototype`キーを拒否しているか(プロトタイプ汚染対策)
- **なぜ重要か**: グローバルObjectプロトタイプの汚染は、無関係なインスタンスや他ユーザーのリクエスト処理にまで波及する。
- **出典**: Web Application Security 2nd Edition 第16章・第34章

### 6. mass assignment対策として更新可能フィールドをallowlist化またはDTOで制限しているか
- **なぜ重要か**: リクエストボディを無検証でDB更新に渡すと、isAdmin等のフィールドまで意図せず書き換えられ権限昇格を許す。
- **出典**: Web Application Security 2nd Edition 第15章・第33章

### 7. アプリケーション用DBアカウントに最小権限を適用し、sysadmin等の過剰権限を与えていないか
- **なぜ重要か**: 過剰権限はSQLi等が発生した際の被害範囲を増幅させる。
- **出典**: Defensive Security Handbook 2nd Edition 第13章 p.138-139, 144

### 8. データベースを安易にインターネットへ公開せず、認証を適切に設定しているか
- **なぜ重要か**: 未認証のまま公開されたデータベースは、攻撃者によるスキャンで即座に発見・悪用される(MongoDBの大量漏洩事例等)。
- **出典**: Defensive Security Handbook 2nd Edition 第13章 p.139

### 9. ユーザー入力をDOMへ挿入する際、`innerHTML`でなく`innerText`/テキストノードを既定にしているか
- **なぜ重要か**: innerHTMLはHTMLとして解釈されるため、サニタイズ漏れがそのままXSSになる。
- **出典**: Web Application Security 2nd Edition 第28章

### 10. CSPの`script-src`をnonce/hashベースのstrict CSPとし、`unsafe-inline`/`unsafe-eval`を有効化していないか
- **なぜ重要か**: stored型/DOM型XSSはCSPだけでは防げず、evalの排除との併用が必須である。
- **出典**: Web Application Security 2nd Edition 第22章・第28章

### 11. サードパーティスクリプトにSubresource Integrity(SRI)のhash検証を付与しているか
- **なぜ重要か**: CDN上のスクリプトが改変された場合でも、ブラウザが実行前に検知できる。
- **出典**: Web Application Security 2nd Edition 第22章

### 12. CORS(Access-Control-Allow-Origin)やCORPヘッダをワイルドカードでなく必要最小限のオリジンに絞っているか
- **なぜ重要か**: 最小権限の原則に基づき、意図しないクロスオリジンからの情報漏洩やSpectre型サイドチャネル攻撃を防ぐ。
- **出典**: Web Application Security 2nd Edition 第22章

### 13. `frame-ancestors`ディレクティブとframebusterスクリプトでクリックジャッキングを防いでいるか(legacyな`X-Frame-Options`だけに依存していないか)
- **なぜ重要か**: 透明なiframeを重ね合わせることでユーザーのクリック操作を乗っ取られる。
- **出典**: Web Application Security 2nd Edition 第16章・第34章

### 14. 新規タブを開く`<a target="_blank">`やwindow.open()に`rel="noopener noreferrer"`を付与し、COOPを`same-origin`に設定しているか
- **なぜ重要か**: `window.opener`経由で元タブ・新タブ双方のページ内容を書き換えるタブナビング(リバースタブナビング)を防ぐ。
- **出典**: Web Application Security 2nd Edition 第16章・第34章

### 15. 状態変更を伴うリクエストをHTTP GETで実装していないか(副作用のあるGETエンドポイントが存在しないか)
- **なぜ重要か**: `<img>`タグ等がユーザー操作なしでGETリクエストを発行でき、それだけでCSRFが成立する。
- **出典**: Web Application Security 2nd Edition 第11章・第29章

### 16. CSRFトークンをリクエスト単位で発行し、「トークンプール」方式(使い回し可能な複数トークンの発行)になっていないか
- **なぜ重要か**: プール方式では正規に発行された別のトークンの転用を許してしまい、防御として不十分になる。
- **出典**: Web Application Security 2nd Edition 第11章

### 17. 全通信をHTTPS化し、HSTSを実装しているか(301リダイレクトのみに依存していないか)
- **なぜ重要か**: 301リダイレクトのみでは初回のHTTPリクエストがMITM攻撃に対して脆弱なままになる。
- **出典**: Web Application Security 2nd Edition 第21章・第22章

### 18. データを保管時・通信時ともに暗号化しているか(AES-256、TLS等)
- **なぜ重要か**: 暗号化は盗難・傍受時の最終防御線となる。
- **出典**: Defensive Security Handbook 2nd Edition 第13章 p.139-143

### 19. HTTP Basic/Digest認証のような資格情報が平文または再送可能な形で扱われる方式を避け、TLS必須のうえでOAuth等のトークンベース認証を選定しているか
- **なぜ重要か**: HTTP Basic認証は通信傍受やXSSにより資格情報が容易に漏洩する。
- **出典**: Web Application Security 2nd Edition 第3章・第5章

### 20. パスワードをBcrypt/PBKDF2等の低速ハッシュ+ソルトで保存し、MD5/SHA1等の高速アルゴリズムを使っていないか
- **なぜ重要か**: 高速アルゴリズムやソルトなしハッシュはレインボーテーブル攻撃・ブルートフォースで容易に解読される。
- **出典**: Web Application Security 2nd Edition 第21章 / Defensive Security Handbook 2nd Edition 第15章 p.176-180

### 21. 高権限アカウント・リモートアクセスにMFAを必須化しているか(push通知よりコード入力型を推奨しているか)
- **なぜ重要か**: MFAはパスワード単体の侵害を無効化できるが、push通知型は「MFA疲労(フラッディング)」攻撃に弱い。
- **出典**: Web Application Security 2nd Edition 第21章 / Defensive Security Handbook 2nd Edition 第15章 p.193 / Identity Security for Software Development 第1・3章

### 22. 認可チェックを一元化されたクラス/コンポーネントで行い、API毎に個別実装していないか
- **なぜ重要か**: 認可ロジックを個別実装すると、人的ミスによる欠落(特定エンドポイントだけ認可漏れ)が発生しやすい。
- **出典**: Web Application Security 2nd Edition 第3章

### 23. 権限変更(解雇・降格等)後、既存の認証トークン/セッションが即座に無効化される継続的認可(Zero Trust)を実装しているか
- **なぜ重要か**: 有効期限の長いトークンが権限変更後も有効なままだと、解雇された従業員等による不正操作が可能になる。
- **出典**: Web Application Security 2nd Edition 第21章

### 24. オブジェクト参照(IDOR)にアクセス権チェックと推測困難なID/ファイル名を用いているか
- **なぜ重要か**: 連番IDは、アクセス権チェックがなければ直接指定で他人のリソースに到達できる。
- **出典**: Web Application Security 2nd Edition 第15章・第33章

### 25. ログインエラー等を汎用メッセージにし、ユーザーの存在/非存在を区別する応答(エラー文言・応答時間差)を返していないか(アカウント列挙対策)
- **なぜ重要か**: 存在確認の応答差は、標的型アカウント乗っ取り攻撃の足がかりになる。
- **出典**: Web Application Security 2nd Edition 第23章

### 26. 認証後のセッションにタイムアウトと適切な失効ポリシーを設けているか
- **なぜ重要か**: 厳格なセッション失効がないと、端末放置やトークン漏洩時の不正アクセスの時間窓が拡大する。
- **出典**: Identity Security for Software Development 第2章

### 27. APIの認証・認可をOWASP API Security Top10に基づきテストしているか(オブジェクトレベル/機能レベルの認可漏れを含む)
- **なぜ重要か**: API認証・認可不備は大規模データ流出や、車両の遠隔始動・解錠などの実被害に直結する(Ferrari/BMW等での研究者による実証例)。
- **出典**: Software Supply Chain Security 第7章 p.120

### 28. JWTを使う場合、有効期限(exp)を必ず設定し、ペイロードを軽量化し、常にAuthZサーバ側で再検証しているか
- **なぜ重要か**: JWTは自己完結型トークンであり、検証を自分のアプリ側の判断だけに頼ると、改ざんされたトークンを受理してしまうリスクがある。
- **出典**: Identity Security for Software Development 第3章

### 29. セキュリティイベント(ログイン失敗、権限変更等)を十分にログ記録し、パスワード等の機密情報を記録から除外しているか
- **なぜ重要か**: セキュリティログ・監視の不備(OWASP Top Tenの「Security Logging and Monitoring Failures」)は、侵害の検知遅延につながる最も一般的な失敗の一つである。
- **出典**: Identity Security for Software Development 第2・4・8章

### 30. ビジネスロジック(残高計算、割引、キャッシュバック、価格、期間限定オファー、割引コード等)に対し「worst-case design」の発想で悪用シナリオ・誤用/濫用ケース(abuse case)を洗い出しているか
- **なぜ重要か**: 正規機能の組み合わせだけで不正な利益を生むことができ(キャッシュバックカード×決済代行の套利事例等)、技術的な脆弱性がなくても被害が成立する。
- **出典**: Web Application Security 2nd Edition 第18章・第36章 / Security Architecture for Hybrid Cloud 第10章

### 31. ビジネスロジック脆弱性の検知にヘッドレスブラウザ等を用いた統計的モデリング/ファジングを導入しているか
- **なぜ重要か**: ビジネスロジック脆弱性は自動ツール(SAST/DAST/SCA)では検出できない領域であり、専用の手法での補完が必要になる。
- **出典**: Web Application Security 2nd Edition 第36章
