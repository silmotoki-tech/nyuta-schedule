# 引き継ぎメモ（nyuta-schedule）

このファイルは、このアプリの開発を新しく引き継ぐ人（AIでも人でも）が最初に読むためのものです。
もっと詳しい経緯は `docs/decisions.md`（設計判断）・`docs/timeline.md`（年表）・`docs/bugs.md`
（不具合と再発防止策）にあるので、作業前に必ず目を通してください。`AGENTS.md` にも同じ指示が
あります。

## このアプリは何か

にゅうた動物病院の院内スケジュール管理アプリ。院内の約10台のiPadで使われ、オペ（手術）・
検査・休み（スタッフ休暇）・研修（実習生など）・診療日（休診日/予約診療日などの状態）・
その他、の6カテゴリの予定・情報をカレンダー形式で登録・共有する。Googleカレンダーより
「1日に入る予定の見やすさ」と「複数端末での同時利用」を優先して独自に作った。

## 構成

- フロントエンド：単一の `index.html`（素のHTML/CSS/JS、ビルド不要）
  - GitHub Pages で公開: https://silmotoki-tech.github.io/nyuta-schedule/
  - リポジトリ: https://github.com/silmotoki-tech/nyuta-schedule （Organization: silmotoki-tech）
- バックエンド：Google Apps Script（`Code.gs`）
  - **病院のGoogle Workspaceアカウント側**で作成されている。個人アカウント側を探さないこと。
  - Googleスプレッドシート「にゅうた動物病院_スケジュールDB」をDB代わりに使う
    - URL: https://docs.google.com/spreadsheets/d/1lj6SRJXYp7w3WfEJMLEB0DaORmQFj6cWzjbEaSG56hc/edit
    - シート名 `events`。列は id / date / category / label / sub / species / time /
      traineeKind / dayoffPeriod / memo / deleted / deletedAt / updatedAt
  - デプロイ: ウェブアプリとして公開（実行するユーザー: 自分 / アクセス: 全員）。
    デプロイURLは `index.html` 内の `GAS_URL` 定数に設定されている。
    Code.gs 自体は**GitHub管理外**（Apps Scriptエディタ側が原本）。
    Code.gs を書き換えたら、Apps Scriptエディタから再デプロイが必要。
  - `LockService` で同時書き込み時の競合を防止している。

## 主な機能

- カテゴリ: オペ／検査／休み／研修／診療日／その他（表示順は `CATEGORY_ORDER`）
- スタッフの複数選択（休みの一括登録、面談の複数人まとめ登録）
- 複数週にわたる繰り返し登録
- オペの術前検査期限リマインダー（🍋。オペの新規登録時にチェックを入れた場合だけ、手術日の
  指定日数前に同じ患者名の別エントリをオペとして作成）
- 絵文字マーキング: 🍎（オペ：チャート準備済み）/ 🍋（オペ：術前検査の期限）/ 🈵（診療日：
  「オペ検査ストップ」の日。2026-10-29追加。入力は止めないが注意書きを出す）
- パスコードロック（初期値 `2211`。1日単位でlocalStorageに保持、管理パネルからログアウト可）
- ソフトデリート（物理削除せず、ゴミ箱扱い。既定60日表示、全件表示切替あり）
- 保存失敗時のローカルフォールバック保存と、次回起動時の復旧・マージ
- 上下スクロールでの無限スクロール（2026-10-26に過去方向の`prependWeeks()`を追加。
  それまでは「今日の2週間前」より過去が見えなかった）

## 現状（2026-10-30時点）

- 本番運用中。院内10台のiPadで安定稼働している（ユーザー評: 「だいぶ安定した運用ができている」）
- 直近の変更は「オペ検査ストップ」マーク（10/29）と、過去スクロール対応（10/26）
- 既知の制約・注意点は `docs/bugs.md` の「技術的な落とし穴」を参照
  （Google Sheetsの日付/時刻自動変換対策、CORSのContent-Type指定など。
  ここを外すと同じ不具合が再発する）

## 作業の進め方（推奨）

- GitHubへの反映はClaude/Cursor側から直接 `git` で作業する（貼り付け方式はフォールバック）
- `index.html` をブラウザで直接編集する場合は部分パッチではなく全文で提示する
- 設計判断・不具合修正をしたら、実装変更と同じタイミングで
  `docs/decisions.md` / `docs/bugs.md` / `docs/timeline.md` に追記する
- Code.gs を変更したら、Apps Scriptエディタ上で再デプロイするまで本番に反映されない
  （GitHub側の `index.html` を更新しただけでは backend は変わらない）

## 困ったときに確認する場所

- 過去の時系列・何が起きたか: `docs/timeline.md`
- なぜその設計にしたか: `docs/decisions.md`
- 同じ不具合を繰り返さないための注意点: `docs/bugs.md`
- スプレッドシートの実データ確認: 上記スプレッドシートURL（病院アカウントでの閲覧権限が必要）