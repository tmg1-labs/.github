# 現在の作業コンテキスト

最終更新: 2026-07-01（4リポジトリともGitHub `tmg1-labs` へpush済み。本リポジトリも公開済み）

## 今やっていること（過去分・組織公開準備）
- **GitHub組織 `tmg1-labs` の作成**（2026-06-29〜）。
  - GitHub組織 `tmg1-labs` を作成済み（個人で Free 組織を作成）。GitLabの `tsumugi` は
    取得済みだったため `tmg1-labs` を採用。
- **組織の `.github` リポジトリ（= 本リポジトリ）を再構成**（2026-06-30、commit `3002954`）。
  - `profile/README.md`: 組織ランディング + 各リポジトリ目次（codec/esp32-demo/cli）を新規作成。
  - `docs/tmg1-format.md`（英）/ `docs/tmg1-format.ja.md`（日）: 仕様書を分離。日本語版を新規作成、
    英版は旧README を整形しテーブル崩れを修復。
  - `SECURITY.md`（英/日、連絡先 `faces@rainorshine.asia`）を新規作成。
  - ルート `README.md` をリポジトリ説明（中身の目次）へ差し替え（旧README の仕様書本文はdocsへ移設）。
  - リンクは `github.com/tmg1-labs/<repo>` 前提で統一。
- **README間クロスリンク方針の決定**: 兄弟リポジトリ一覧のベタ書きは保守コスト
  （リポジトリ追加・改名で全README を修正する羽目になる）が高いため、一覧の正本は
  組織トップ（本リポジトリの `profile/README.md`）に集約し、各リポジトリREADMEには機能リンク
  （依存先codec / 仕様書 / 入力生成cli）＋「Part of TMG1 Labs」固定ポインタのみ残す方針に決定。
  各リポジトリのREADMEへこの方針を反映済み（`tmg1-codec` commit `0855972`、`tmg1-cli` commit
  `73e52f3`、本リポジトリ commit `c096b4f`）。
- **リポジトリ改名**: `tmg1-arduino` → `tmg1-esp32-demo`、`gitlab-profile`（ローカルフォルダ名） →
  `tmg1-labs.github`（実体は `tmg1-labs/.github`）。GitHub公開・remote切替・README整備は完了。

## 一時的な制約・注意事項
- なし（GitHub公開・4リポジトリのpushは完了済み）。

## 次にやること
- 特になし。新リポジトリを追加する場合は `workflows.md` の公開順序制約に従うこと。

## 参考
- 決定経緯・セッション履歴の詳細は `session-history.md`、過去のエラー解決は `errors-log.md`。
