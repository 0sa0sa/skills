# 適用事例: NousResearch/hermes-agent (2026-11-02)

> この文書は `ai-agent-review` スキルを実在のOSSに適用した結果の記録であり、スキル本体(`SKILL.md` / `references/`)からは参照されていない。スキルの動作には影響しない。

## 対象

- リポジトリ: [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)
- 概要: 自己改善AIエージェント runtime。MITライセンス、約25万スター。Python中心(約118MBのPythonコード)+TypeScript(デスクトップアプリ)。
- 特徴: 非常に成熟した `SECURITY.md`(脅威モデル、トラストバウンダリ、スコープ外項目を明文化)を公開している。

## 方法

リポジトリ全体(21,976ファイル)は規模的に網羅不可能なため、以下9領域に絞って調査した。各領域は「抽出(Review)→検証(Verify、懐疑的な別エージェントが実ファイルを再読して確認)」の2段パイプラインで実施。

| 領域 | 対応カテゴリ | 対象ファイル数 | 確認された発見 |
|---|---|---|---|
| 承認ゲート(approval gate) | safety-guardrails | 5 | 2 |
| 認証情報スコーピング | safety-guardrails | 3 | 0 |
| Skills Guard / プラグイン審査 | safety-guardrails | 4 | 0 |
| 外部サーフェスの認可(allowlist) | safety-guardrails | 4 | 0 |
| ツール実行基盤(terminal backend / code execution) | tool-use-execution | 4 | 1 |
| サブエージェント委任(async delegation) | agent-architecture | 4 | 2 |
| リトライ・フォールバック・エラー処理 | reliability-recovery | 4 | 1 |
| コンテキスト圧縮・トラジェクトリ圧縮 | performance-cost | 3 | 1 |
| プロンプト再構築 | prompt-context | 1 | 0 |

**扱わなかった領域**: rag-retrieval(古典的RAGが存在しないため対象外)、evaluation-monitoring、ux-human-ai、operations-lifecycle。規模上の判断であり、「問題なし」という評価ではない。

安全性関連の4領域では、対象プロジェクト自身の `SECURITY.md` を各レビューエージェントに読ませ、「承認ゲート・出力redaction・Skills Guardはヒューリスティックであり境界ではない」「それらのバイパースはそもそも脆弱性として扱わない」という同プロジェクトの明言済みスコープを踏まえ、ヒューリスティックのバイパース報告(対象外)ではなく、ドキュメントの約束と実装の不一致・内部ロジックの矛盾を探すよう指示した。

## 確認された発見(7件)

### [high] `tools/approval.py:1256` — 承認ゲート間の矛盾によるfail-open
cron/single_query の複数承認コンテキストが同時に成立する場合、`check_execute_code_guard` と `_run_approval_gate` は最初の1コンテキストのみで判定し、2番目以降(cron, mode=deny)を評価しない。一方 `check_all_command_guards` は全コンテキストを見てdenyを優先する。同一ファイル内の「正式な」ゲート関数同士が矛盾した判定を返し、`cron_mode: deny` を設定していても `execute_code` が無承認実行される経路がある。

### [high] `tools/async_delegation.py:857` — 委任レコードのリーク
`_dispatch_admitted` は `_records` に `running` 状態を書き込んだ直後、保護なしで `_persist_dispatch(record)` を呼ぶ。これが例外を投げると(ワーカー経由のpid不一致、SQLiteロックタイムアウト等)、`executor.submit` は一度も呼ばれないのにレコードは `running` のまま永久に残り、`_sweep_stale_locked` は `_started=False` のレコードを永久にスキップするため回収されない。直後にある `executor.submit` 失敗時の対称的なクリーンアップがここには欠けている。

### [medium] `agent/api_error_summary.py:218` — エラー要約の一部経路でredaction漏れ
`_summarize_api_error` は3つの戻り経路(`error.body`, `response.text`, 最終rawフォールバック)のうち `response.text` 経路だけ `redact_sensitive_text()` を通す。残り2経路は同じコメントが「攻撃者/プロバイダ由来でAuthorizationやAPIキーを含みうる」と明言しているにもかかわらずredactionなしでそのまま返す。

### [medium] `tools/approval.py:856` — 人間の拒否がLLM専用カウンタを誤加算
`_human_decision` のtransport経由の人間deny分岐だけが `_record_denial()` を呼び、「連続DENY判定はガーディアンLLMのものだけを数える」という設計(CLI/gateway分岐のコメントで明言)に反する。結果、人間が3回(デフォルトしきい値)transport経由で拒否するだけで「セキュリティレビュアーがブロックした」という実際と異なるハードストップ文言が返る。

### [medium] `tools/code_execution_rpc.py:248` — RPC応答書き込みの未検証
`_rpc_poll_loop` 内のレスポンス書き込みが `_remote_write(..., check=False)` のままで成否未検証。同ファイルの別の書き込み(`_ship_file_to_remote`)は `check=True` で明示的に検証しており非対称。失敗時にツール呼び出しが二重実行される、または偽のRPCタイムアウトが報告される。

### [medium] `tools/async_delegation.py:177` — ワーカー経由委任のフォレンジック欠落
オーナープロセス直接のディスパッチでは `task_transcripts` / `owner_cwd` が永続化され、異常終了時の `recover_abandoned_delegations` がフォレンジック情報を生成できる。しかし `WorkerDelegationLedger.dispatch`(ワーカー経由のネスト委任)が組み立てるペイロードにはこの2フィールドが含まれず、同じ復旧機能がネストした委任にだけ効かない。

### [medium] `trajectory_compressor.py:332` — HTTPクライアントの再生成
`_get_async_client` は呼び出しごとに新しい `AsyncOpenAI` クライアント(独自のhttpxコネクションプール付き)を生成し、既存のクライアントは `aclose()` されずに捨てられる。`max_concurrent_requests=50` のデフォルトで数千件のトラジェクトリを処理すると、本来再利用できるはずのコネクションプールを使わず毎回TCP/TLSハンドシェイクが発生し、ソケット/FDが蓄積する。

## 「発見なし」だった領域について

認証情報スコーピング・Skills Guard/プラグイン審査・外部サーフェスのallowlistの3領域では、検証エージェントが具体的な指摘を確認できなかった(=その実装がSECURITY.mdの約束と一致していると判断した)。これは「レビューが機能しなかった」のではなく、該当実装がドキュメント通りに実装されている可能性を示す結果として記録する。ただし、レビューエージェントは最大数個のファイルしか読んでおらず、これをもって「この3領域に問題がない」と断定するものではない。

## スキルの有効性についての所感

- 書籍4冊から抽出したチェックリストは、大規模で成熟したOSS(自前の脅威モデル文書を持つプロジェクト)に対しても、具体的で検証可能な指摘(関数名・行番号付き)を生成できた。
- 対象プロジェクト自身の `SECURITY.md` を読ませてスコープを調整する(ヒューリスティックのバイパース報告を除外する)工夫が、的外れな指摘を避けるのに有効だった。
- 「抽出→懐疑的検証」の2段構成により、検証段階で2件(prompt-context, tool-execution領域それぞれ1件)がREJECTされ、精度向上に寄与した。
- 今回の規模(9領域・約50ファイル)でも約160万トークンを要した。リポジトリ全体への適用にはさらに大きなスケーリングが必要。
