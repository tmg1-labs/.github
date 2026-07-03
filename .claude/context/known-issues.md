# 既知の問題・注意事項

> 過去のエラー解決の原文は `errors-log.md`。ここはハマりやすい地雷の要点を集約する。

## ドキュメント

### `profile/README.md` はサブディレクトリ配下＋Organizationプロフィール表示の二重の制約を受ける
- **内容**: `profile/README.md` はリポジトリ内では `profile/` というサブディレクトリに置かれる
  一方、GitHubの仕様で `.github` リポジトリの `profile/README.md` は Organization のプロフィール
  ページとしてもそのまま表示される特殊ファイル。
- **地雷1（相対パスの基準）**: 通常のリポジトリ閲覧（blob表示）では相対リンクはファイル自身の
  ディレクトリ基準で解決される。`profile/README.md` から `docs/tmg1-format.md` と書くと
  `profile/docs/tmg1-format.md` を指してしまい404になる（正しくは `../docs/tmg1-format.md`）。
- **地雷2（プロフィールページでの相対リンク破綻）**: GitHub公式ドキュメントによれば、
  Organizationプロフィールページ上では相対リンク・相対画像パスが正しく解決されない。
- **回避策**: `profile/README.md` 内のリンクは相対パスではなく
  `https://github.com/tmg1-labs/.github/blob/main/...` のような絶対URLで書く
  （2026-07-03、`docs/tmg1-format.md`/`ja.md` へのリンクで実際に発生・修正）。

## リポジトリ公開順序の制約

### 相対URL submoduleの解決はsuperprojectのoriginに依存する
- **内容**: `tmg1-cli` の `.gitmodules` は相対URL（`../tmg1-codec.git`）で `tmg1-codec` を
  参照している。この相対URLはGitHub上ではsuperproject（`tmg1-cli`）のorigin
  （`tmg1-labs/tmg1-cli.git`）に対して解決されるため、実際には `tmg1-labs/tmg1-codec.git` を
  指すことになる。
- **含意**: `tmg1-codec` がまだGitHub上に存在しない状態で `tmg1-cli`/`tmg1-esp32-demo` を先に
  公開すると、submodule/CI の checkout が解決できず失敗する。**必ず `tmg1-codec` を先に公開**
  してから依存側を公開すること。
- 新しくリポジトリを追加する場合も同じ順序制約に注意（`workflows.md` 参照）。
