# 既知の問題・注意事項

> 過去のエラー解決の原文は `errors-log.md`。ここはハマりやすい地雷の要点を集約する。

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
