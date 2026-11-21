# ハムスター手帖 引継ぎ (2026-11-12)

- 目的: ハムスターの部屋掃除・大掃除の「前回」「次回」を管理。手帖ポータルのカードから開く。
- 構成: 単一 index.html（GitHub Pages想定、リポジトリ名 hamster-techo）。データは localStorage(hamsterdb.v1) + Google Drive(hamster-db.json) 同期。Drive同期は他の手帖と同じ方式・同じOAuthクライアントID。
- 仕様:
  - 次回日: 部屋掃除=前回+7日、大掃除=前回の翌月の第1木曜日（設定で変更可）。手動で上書き可（記録すると自動に戻る）。
  - 大掃除をした日は部屋の掃除も済み扱い（設定でON/OFF）。
  - 通知なし。「カレンダーに登録」ボタンでGoogleカレンダーAPIから自動登録（時間つき可、既存予定は更新）。名称は「こめちの家の掃除」「こめちの家の大掃除」。
  - 初期値: 部屋掃除 2026-11-12、大掃除 2026-11-02。
- タブ: ホーム / 履歴 / カレンダー / 設定。
- 未了: GitHubリポジトリ作成・Pages公開、ポータル2か所（GitHub portal と claude.aiアーティファクト）にカード追加。
- カレンダー自動登録の前提（GCPの共有OAuthクライアント 568813131190…）: ① Google Calendar API を有効化 ② 承認済みリダイレクトURIに https://ykdtmnhnddsbba-jj.github.io/hamster-techo/ を追加（Drive同期にも必要）。初回登録時に calendar.events の同意画面が出る。
- 時間: 掃除の記録・次回日の手動修正で時刻を入れられる。設定に既定時刻と予定の長さ（既定60分）。


## 共通Googleログイン（2026-11）
- Googleログインと Drive 同期の処理は `../portal/lib/techo-google.js`（portalリポジトリ）に1本化した。各手帖は `TechoGoogle.create({site,...})` を呼ぶだけ。
- トークンは同じオリジン（ykdtmnhnddsbba-jj.github.io）の localStorage `techo_gauth_tok` に共有で保存。どれか1つの手帖でログインすれば、他の手帖も認証画面なしで同期できる。
- 同期は `GA.syncFile(ファイル名, id, マージ関数, 本文関数)`。Drive の読み込みに失敗したら上書きせずに中止する（データ消失防止）。ファイルが404なら id を忘れて次回検索し直す。
- portal の lib が読めない時もページ自体は表示される（同期だけ無効）。

## HOME✦MISSION 連携（2026-11）
- 設定タブ「📣 HOME✦MISSION で通知」。次回の掃除日を kazoku-todo の「1回だけ」ミッション（id `ham-room` / `ham-big`）として登録・更新する。
- 接続先とカギは HOME✦MISSION がこの端末に保存した localStorage `kazoku.cfg`（{api,key}）を読む（同じ github.io なので読める）。この端末で HOME✦MISSION を一度開いていないと設定できない。
- オン/オフ・担当・完了の条件・既定の時刻は `db.config.mission` に入れて Drive で同期。何を送ったかは端末ごとに `hamsterdb.mission.v1` に覚えて、同じ内容は送り直さない。
- 次回日が過ぎていたら「今日」で登録する（過去の日付だと通知されないため）。
- 起動時に HOME✦MISSION のミッション日付を読み、そこで完了されていれば「HOME✦MISSIONで完了（名前）」として掃除の記録を追加する（前回の記録がその日以降なら追加しない）。
- オフにすると2つのミッションを削除する。
