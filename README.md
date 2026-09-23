# sparring

壁打ち用の Agent Skills セット。サービス企画・ゲームデザイン・小説プロットの3領域について、専門家役と1問ずつ対話しながら企画を詰める。SKILL.md 形式に対応した Claude Code / Codex / OpenCode / Kiro CLI / Antigravity CLI で動く。

## 構成

```
skills/       スキル本体（各ツールにインストールするのはここだけ）
  sparring-service/   サービス企画の壁打ち（引数: 業種）
  sparring-game/      ゲームデザインの壁打ち（引数: ジャンル）
  sparring-plot/      小説プロットの壁打ち（引数: エージェント名）
agents/       役割定義（25本。スキルが実行時に読み込む）
  planner-*.md        業種別のサービス企画（EC／ヘルスケア／金融／教育／人材／ゲーム／飲食／旅行／不動産／コミュニティ／メディア／地域公共／BtoB SaaS）
  service-planner.md  サービス企画の汎用役
  gamedesigner-*.md   ジャンル別のゲームデザイン（ライフシム／アクション／RPG／経営シム／パズル／マルチプレイ）
  game-designer.md    ゲームデザインの汎用役
  novelist-*.md       ジャンル別の小説家（ライトノベル／一般文芸／ミステリ／SF・ファンタジー）
common-persona/  共通規範（役割定義が最初に読み込む）
  planner.md          サービス企画の共通規範
  gamedesigner.md     ゲームデザインの共通規範
  novelist.md         小説の共通規範
```

スキルは引数から `agents/` の役割定義を選んで Read し、その定義が指す共通規範も Read して、会話の中でその役を演じる。サブエージェントは使わないので、`agents/` を各ツールのエージェントとして登録する必要はない。

## インストール

### 1. リポジトリを clone する

スキルと役割定義は `~/.myai/sparring/...` の固定パスで互いを参照しているので、この場所に置く。

```sh
git clone https://github.com/sfukuda84/ai-sparring.git ~/.myai/sparring
```

### 2. 使うツールのスキルディレクトリにリンクする

既存のスキルを残すため、ディレクトリごとではなくスキル単位でシンボリックリンクする。

```sh
link_skills() {
  mkdir -p "$1"
  for s in "$HOME"/.myai/sparring/skills/*/; do
    s=${s%/}; ln -sfn "$s" "$1/$(basename "$s")"
  done
}
```

| ツール | リンク先 | コマンド |
|---|---|---|
| Claude Code | `~/.claude/skills/` | `link_skills ~/.claude/skills` |
| Codex CLI | `~/.agents/skills/` | `link_skills ~/.agents/skills` |
| OpenCode | `~/.claude/skills/` か `~/.agents/skills/` を自動で読む | 上のどちらかを実行済みなら不要 |
| Kiro CLI | `~/.kiro/skills/` | `link_skills ~/.kiro/skills` |
| Antigravity CLI | `~/.gemini/antigravity-cli/skills/` | `link_skills ~/.gemini/antigravity-cli/skills` |

Claude Code で `CLAUDE_CONFIG_DIR` を使って設定ディレクトリを分けている場合は、`link_skills "$CLAUDE_CONFIG_DIR/skills"` のように各ディレクトリで実行する。スキルを追加したら、再実行してリンクを足す。

## 使い方

| ツール | 呼び出し方 |
|---|---|
| Claude Code | `/sparring-service ec 犬用おやつの D2C ブランドを考えています` |
| Codex CLI | `$sparring-service ec 犬用おやつの D2C ブランドを考えています` |
| OpenCode | `sparring-service スキルを使って、業種 ec で壁打ちしたい。犬用おやつの D2C ブランドを考えています`（スラッシュコマンドにはならないので、スキル名を文中で指定する） |
| Kiro CLI | 対話モードで `/sparring-service ec ...`。`--no-interactive` ではスラッシュコマンドが効かないので、OpenCode と同じく文中でスキル名を指定する |
| Antigravity CLI | `/sparring-service ec ...`（リポジトリを読むために `--add-dir ~/.myai/sparring` を付けて起動する） |

他のスキルも同じ形で呼ぶ。

```
/sparring-game rpg 成長が数値の水増しになりがちな JRPG を作っています
/sparring-plot novelist-mystery 雪山の山荘で起きる密室殺人のプロットを相談したい
```

引数は日本語や略称でもよい（例: `飲食`、`d2c`、`ローグライト`）。省略すると、企画の内容から推定して一言確認してから始める。

## 注意

- 別の場所に clone する場合は、`skills/*/SKILL.md` と `agents/*.md` の中の `~/.myai/sparring` を書き換えること。
- ワークスペース外のファイル読み取りを制限するツールでは、`~/.myai/sparring` を読めるようにしておく（Antigravity CLI の `--add-dir`、OpenCode の `external_directory` 権限など）。
- 役割定義や規範ファイルを読めなくても、モデルはエラーのまま壁打ちを続けてしまい、見た目では気づけない。変更後は非対話モードで実行し、`agents/` と規範ファイルを読んでいるか確認するとよい（例: `claude -p "/sparring-service ec ..." --output-format stream-json --verbose`）。
