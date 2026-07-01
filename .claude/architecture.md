# アーキテクチャ方針

## 全体構成

GitHub組織 `tmg1-labs` の `.github` リポジトリ。コードは含まない、ドキュメント専用リポジトリ。

```
tmg1-labs.github (このリポジトリ = tmg1-labs/.github)
├── profile/README.md         ... 組織ランディングページ（github.com/tmg1-labs に表示）
├── docs/tmg1-format.md       ... TMG1フォーマット仕様書（英語・正本）
├── docs/tmg1-format.ja.md    ... TMG1フォーマット仕様書（日本語）
├── SECURITY.md                ... 組織全体のセキュリティポリシー
└── README.md                  ... 本リポジトリ自体の説明（目次）
```

## 各リポジトリREADMEとのリンク方針（設計決定）

各リポジトリ（codec/cli/esp32-demo）のREADMEに兄弟リポジトリの一覧を書く方式は、
リポジトリの追加・改名のたびに全README を修正する羽目になり保守コストが高い。
そのため以下の方針を採用している:

- 兄弟リポジトリ一覧・役割説明の**正本は本リポジトリの `profile/README.md` に一本化**する。
- 各リポジトリのREADMEには「機能リンク（依存先codec / 仕様書 / 入力生成cli 等）」＋
  「Part of TMG1 Labs」固定ポインタのみを残す。
- フォーマット仕様書（`docs/tmg1-format(.ja).md`）も本リポジトリに集約し、各リポジトリの
  README からはリンクのみとする（実装側READMEにバイト単位の仕様を重複記載しない）。

## 公開順序の制約

各リポジトリの `.gitmodules` は相対URL（`../tmg1-codec.git` 等）を使っている。これは
GitHub上ではsuperprojectのorigin（`tmg1-labs/xxx.git`）に対して解決されるため、
**`tmg1-codec` を先にGitHub公開してから `tmg1-cli`/`tmg1-esp32-demo` を公開する**という
順序制約がある（詳細は `workflows.md` / `known-issues.md`）。今後リポジトリを追加する際も
同様の順序制約に注意すること。

## 禁止パターン

- 各リポジトリのREADMEに兄弟リポジトリ一覧をベタ書きしない（本リポジトリへのリンクのみ）。
- フォーマット仕様のバイト単位の詳細を実装側READMEに重複させない。
