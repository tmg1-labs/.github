# ドキュメント執筆規約

## 英日併記
- 英語版が正（`docs/tmg1-format.md` 等）、日本語版（`*.ja.md`）を並行して整備する。
- 冒頭で相互にリンクする（英語版から日本語版へ、日本語版から英語版へ）。

## クロスリンク
- 兄弟リポジトリの一覧・役割説明は `profile/README.md` に一本化する
  （`architecture.md` の設計決定を参照）。
- 各リポジトリのREADMEフッタには「Part of [TMG1 Labs](https://github.com/tmg1-labs)」の
  固定ポインタを置く。

## 表記
- リンクは `github.com/tmg1-labs/<repo>` 形式で統一する。
- Markdownテーブルは崩れやすいので、セル内に長い説明を詰め込みすぎない（列を増やすか、
  箇条書きに逃がす）。

## セキュリティ連絡先
- `SECURITY.md` の連絡先は `faces@rainorshine.asia`（英日とも同一）。
