# tmg1-labs / .github

@.claude/architecture.md
@.claude/coding-style.md
@.claude/workflows.md
@.claude/context/current-sprint.md
@.claude/context/known-issues.md

## Quick facts
- コードは無い、GitHub組織 `tmg1-labs` の組織プロフィール + TMG1フォーマット仕様書の正本リポジトリ。
- `profile/README.md`: 組織ランディングページ（`github.com/tmg1-labs` に表示）。各リポジトリの
  一覧・役割・仕様書へのリンクを集約する場所。
- `docs/tmg1-format.md` / `docs/tmg1-format.ja.md`: TMG1フォーマット仕様書（英語が正、日本語版併記）。
- `SECURITY.md`: 組織全体に適用するセキュリティポリシー。

## 関連リポジトリ
- `tmg1-codec`: C++17 コーデック本体。
- `tmg1-esp32-demo`: ESP32ファームウェア（C++/PlatformIO）。
- `tmg1-cli`: Rust製CLI（ffmpegパイプライン→`.tmg1`）。

## Claudeへの指示
- 方針の決定や修正に関する意図や経緯があれば記録していくこと。
- 実装（`tmg1-codec`等）とフォーマット仕様に差異が見つかった場合は、実装を正として
  `docs/tmg1-format(.ja).md` を更新する。
- 各リポジトリのREADMEからの兄弟リポジトリ一覧はここ（`profile/README.md`）に一本化する方針
  （保守コスト回避）。各READMEには機能リンク＋「Part of TMG1 Labs」固定ポインタのみを置く。
- セッションの記録は `session-record` スキルを使う。
- 長期の決定経緯・セッション履歴は `.claude/context/session-history.md`、過去のエラー解決は
  `.claude/context/errors-log.md` を参照。
