# セッション履歴・決定経緯

> 各セッションの作業内容と決定事項のログ。新しいセッションは `session-record` スキルで追記する。

---

### 2026-06-29〜2026-06-30 セッション（GitHub組織公開準備）

#### 作業内容
TMG1エコシステム4リポジトリ（codec/cli/esp32-demo/本リポジトリ）のGitHub公開に向けた
組織プロフィール・仕様書・READMEの整備。

#### 完了したこと
- GitHub組織 `tmg1-labs` を作成（個人でFree組織を作成。GitLabの `tsumugi` は取得済みだったため
  `tmg1-labs` を採用）。
- 本リポジトリ（組織の `.github`）を再構成:
  - `profile/README.md`: 組織ランディング + 各リポジトリ目次（codec/esp32-demo/cli）を新規作成。
  - `docs/tmg1-format.md`（英）/ `docs/tmg1-format.ja.md`（日）: 仕様書を分離。日本語版を新規作成、
    英版は旧READMEを整形しテーブル崩れを修復。
  - `SECURITY.md`（英/日、連絡先 `faces@rainorshine.asia`）を新規作成。
  - ルート `README.md` をリポジトリ説明（中身の目次）へ差し替え（旧READMEの仕様書本文はdocsへ移設）。
- リポジトリ改名: `tmg1-arduino` → `tmg1-esp32-demo`（ESP32専用デモという実態へ寄せるため。
  旧名はプレイヤー主体に見える）。取り込むArduinoライブラリの位置づけを明確化
  （`tmg1-codec` 単体で完結。esp32-demoは「codecを実機で動かすESP32リファレンス」に過ぎない）。

#### 決定事項
| 決定 | 内容 | 理由 |
|---|---|---|
| クロスリンク方針 | 兄弟リポジトリ一覧は組織トップ（本リポジトリ `profile/README.md`）に集約。
  各リポジトリREADMEは機能リンク＋「Part of TMG1 Labs」固定ポインタのみ | 兄弟リストのベタ書きは
  リポジトリ追加・改名のたびに全README修正が必要になり保守コストが高いため |
| リポジトリ名 | `tmg1-arduino`→`tmg1-esp32-demo` | ESP32専用デモという実態を名前に反映 |

#### 注意点
- 公開順序は相対URL submodule解決の都合上 `tmg1-codec` → `tmg1-cli`/`tmg1-esp32-demo` の順が
  必須（known-issues.md参照）。
- 4リポジトリとも当時は未push。2026-07-01に全リポジトリのpushが完了している
  （各リポジトリのcurrent-sprint.md参照）。
