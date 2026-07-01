# よく使うコマンド・手順

## 仕様書の更新
実装（主に `tmg1-codec`）を変更してフォーマットに影響する場合は、必ず
`docs/tmg1-format.md`（英語・正本）と `docs/tmg1-format.ja.md`（日本語）の両方を更新する。
実装が正、仕様書は実装に追随させる。

## 新リポジトリを追加する場合の公開順序
相対URL submodule（`../tmg1-codec.git` 等）はGitHub上でsuperprojectのoriginに対して解決される。
そのため:
1. 依存される側（例: `tmg1-codec`）を先にGitHub公開する。
2. それを submodule/lib_deps で参照する側（`tmg1-cli`, `tmg1-esp32-demo`）を後から公開する。
3. `profile/README.md` のリポジトリ一覧テーブルに新リポジトリを追記する。

## 組織プロフィールの更新
`profile/README.md` はGitHub組織 `tmg1-labs` のランディングページに直接表示される。
リポジトリの追加・改名・役割変更があれば、まずここを更新する（各リポジトリREADME側は
「Part of TMG1 Labs」ポインタのみのため個別修正は不要）。
