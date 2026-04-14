# project-template

Minimal Python project template with [pixi](https://pixi.sh), [Claude Code](https://claude.ai/code), and [Codex CLI](https://openai.com/index/introducing-codex/) cross-model workflow.

## Prerequisites

- [nanokit](https://github.com/DenDen047/nanokit) -- dev environment (pixi, shell, CLI tools)
- [claude-settings](https://github.com/DenDen047/claude-settings) -- Claude Code MCP configuration
- [Codex CLI](https://openai.com/index/introducing-codex/) -- OpenAI's coding agent (`pixi global install codex`)

## Setup

1. Create a new repo from this template on GitHub, then clone it
   ```bash
   git clone https://github.com/<you>/<your-project>.git
   cd <your-project>
   ```
2. **Terminal 1** で Claude Code を起動し、pixi 環境を生成する
   ```bash
   claude
   > /pixi-env
   ```
   このステップで `pixi.toml` が生成される。以降のコマンド・hooks はすべて pixi 環境に依存する。
3. Install dependencies
   ```bash
   pixi install
   ```
4. **Terminal 2** で Codex CLI を起動して動作確認
   ```bash
   codex
   ```

> **Note**: このテンプレートに `pixi.toml` は含まれていない。`/pixi-env` スキルがプロジェクトに合わせた `pixi.toml` を生成する。pixi 環境が未構築の間、Claude Code と Codex CLI の hooks はサイレントスキップする。

## Usage

```bash
pixi run test      # Run tests
pixi run lint      # Lint check
pixi run format    # Auto-format
```

## Design Philosophy

### Why this template exists

Claude Code は強力だが、プロジェクトごとに設定をゼロから組むのは非効率。このテンプレートは「Claude Code と Codex CLI を併用する Python プロジェクト」の最小構成を提供する。

### Two-layer architecture

設定は **グローバル層** と **プロジェクト層** の2層で構成される。

```
~/.claude/           (global -- nanokit が管理)
├── settings.json    # 全プロジェクト共通のパーミッション
├── rules/           # 言語横断のコーディング規約
├── agents/          # 汎用エージェント (planner, tdd-guide, ...)
└── skills/          # 汎用スキル (/pixi-env, /ai-code-changes, ...)

.claude/             (project -- このテンプレートが管理)
├── settings.json    # プロジェクト固有のパーミッション + hooks
├── rules/           # プロジェクト固有の制約 (pixi-only)
├── agents/          # プロジェクト固有のエージェント
├── skills/          # プロジェクト固有のスキル (/check, /plan)
└── hooks/           # ライフサイクルフック
```

グローバル層は「どのプロジェクトでも共通のルールとツール」、プロジェクト層は「このプロジェクト固有の制約と自動化」を担う。Claude Code はセッション開始時に両方を読み込み、プロジェクト層がグローバル層を上書きする。

### Separation of concerns

`.claude/` 配下の各ディレクトリは役割が異なる:

| Directory | Role | Timing |
|-----------|------|--------|
| `rules/` | **制約を宣言する** -- Claude に「何をすべきか/すべきでないか」を伝える | 常時ロード |
| `hooks/` | **品質を自動で担保する** -- ファイル編集後の自動フォーマット、コミット前のリントゲートなど | イベント駆動 |
| `skills/` | **操作手順を定義する** -- ユーザーが `/check` や `/plan` のように明示的に呼び出す | オンデマンド |
| `agents/` | **専門家を派遣する** -- 特定の知識・ツールセットを持つサブエージェント | オンデマンド |

Rules は「法律」、Hooks は「自動取締」、Skills は「マニュアル」、Agents は「専門チーム」と考えるとわかりやすい。

### Cross-model verification

Claude Code (Anthropic) と Codex CLI (OpenAI/GPT) の2つの異なるモデルアーキテクチャを **デュアルターミナル** で併用する。単一モデルでは見落としやすいバイアスや盲点を、別アーキテクチャのモデルで検証する「Two-Key」パターン:

```
Terminal 1 (Claude Code)          Terminal 2 (Codex CLI)
─────────────────────            ─────────────────────
計画 (PLAN)                       QA レビュー (QA REVIEW)
実装 (IMPLEMENT)                  最終検証 (VERIFY)
```

- **Step 1 PLAN**: Claude Code で `/plan` を実行し、フェーズ分割された計画を `plans/` に出力
- **Step 2 QA REVIEW**: Codex CLI で計画をレビューし、見落としを "Codex Finding" として追記
- **Step 3 IMPLEMENT**: Claude Code (新セッション) で phase-by-phase 実装
- **Step 4 VERIFY**: Codex CLI で実装を計画と照合し、乖離を検出

### pixi as the single source of truth

環境管理を pixi に一元化することで:
- `pip install` / `conda` / `venv` の混在による環境の壊れを防ぐ
- `pixi.toml` だけ見れば依存関係がすべてわかる
- Hooks も `pixi run ruff ...` 経由で実行するため、ツールチェインが統一される
- pixi 環境が未構築の段階ではフックはサイレントスキップするので、テンプレート直後でもエラーにならない

## Structure

```
├── CLAUDE.md                    # Claude Code project instructions
├── .claude/                     # Claude Code configuration
│   ├── settings.json            # Team-shared permissions + hooks
│   ├── hooks/
│   │   ├── post-edit-python.sh  # Auto-format & lint after Write/Edit
│   │   └── pre-commit-lint.sh   # Lint gate before git commit
│   ├── agents/
│   │   └── project-reviewer.md  # Code review agent (pixi-aware)
│   ├── skills/
│   │   ├── check/SKILL.md       # /check -- run all quality checks
│   │   └── plan/SKILL.md        # /plan -- create phased plan for cross-model review
│   └── rules/
│       └── pixi.md              # pixi-only package management rules
├── AGENTS.md                    # Codex CLI project instructions
├── .codex/                      # Codex CLI configuration
│   ├── config.toml              # Codex CLI settings
│   ├── hooks.json               # Hook registration (SessionStart, Stop)
│   └── hooks/
│       ├── run-hook.sh          # Wrapper (silent skip if pixi not set up)
│       ├── scripts/hooks.py     # Hook handler (context injection + sound)
│       ├── config/              # Hook enable/disable config
│       ├── sounds/              # Sound files (user-provided, see below)
│       └── logs/                # Hook event logs (git-ignored)
├── plans/                       # Cross-model workflow plan files
├── .gitignore                   # Python + pixi
├── src/                         # Source code
└── tests/                       # Tests
```

`pixi.toml` is not included -- generate it per project with the `/pixi-env` skill or `pixi init`.

## Claude Code Workflow

### Skills & Agents (project-level)

| Type | Name | Purpose |
|------|------|---------|
| Skill | `/check` | Run all quality checks (test + lint + format) |
| Skill | `/plan` | Create a phased plan in `plans/` for cross-model review |
| Agent | `project-reviewer` | Review code for project conventions and pixi compliance |

### Hooks (automatic)

| Event | Script | Behavior |
|-------|--------|----------|
| PostToolUse (Write/Edit) | `post-edit-python.sh` | Auto-format + lint report on `.py` files (async) |
| PreToolUse (Bash) | `pre-commit-lint.sh` | Block `git commit` if lint errors exist (sync) |

Hooks run via pixi (`pixi run ruff ...`). They silently skip if the pixi environment is not set up yet.

### Skills (from nanokit)

| Phase | Skill | Description |
|-------|-------|-------------|
| Environment setup | `/pixi-env` | Initialize `pixi.toml` with conda/PyPI dependencies |
| Coding | `/ai-code-changes` | Pre-change investigation and self-review |
| Coding | `/code-comments` | Write meaningful comments (Why, not What) |
| Review | `/code-review-etiquette` | Actionable review with blocking/nit distinction |
| Commit | `/git-commits` | One logical change per commit, Problem -> Solution message |
| Documentation | `/readme-writing` | Funnel-structured README updates |

## Cross-Model Workflow

### Overview

2 つのモデルアーキテクチャ (Claude + GPT) を交互に使い、単一モデルでは見落としやすいバイアスや盲点を検出する 4 ステップサイクル。

```
  ┌──────────────────────┐    ┌──────────────────────┐
  │   Terminal 1         │    │   Terminal 2         │
  │   Claude Code        │    │   Codex CLI          │
  │                      │    │                      │
  │ Step 1: PLAN ────────┼───→│ Step 2: QA REVIEW    │
  │                      │    │                      │
  │ Step 3: IMPLEMENT ←──┼────│                      │
  │                      │    │ Step 4: VERIFY       │
  │ commit & push        │    │                      │
  └──────────────────────┘    └──────────────────────┘
```

### Terminal Setup

#### Recommended: Dual Terminal

2 つのターミナルペインを横に並べる。

```
┌──────────────────────────┬──────────────────────────┐
│  Terminal 1              │  Terminal 2              │
│  Claude Code             │  Codex CLI              │
│                          │                          │
│  $ claude                │  $ codex                │
│                          │                          │
│  Step 1 (PLAN)           │  Step 2 (QA REVIEW)     │
│  Step 3 (IMPLEMENT)      │  Step 4 (VERIFY)        │
└──────────────────────────┴──────────────────────────┘
```

**Setup commands:**

| Terminal | Tool |
|----------|------|
| iTerm2 | `Cmd+D` で縦分割 |
| tmux | `tmux new -s dev` → `Ctrl+B %` で縦分割 |
| Ghostty | ネイティブ分割 |

**Do NOT use** integrated IDE terminals (VS Code, Cursor) -- full-featured terminal emulators provide better multi-pane support.

#### Advanced: Agent Teams

大規模タスクで複数の Claude Code エージェントを並行稼働させる場合:

```bash
CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 claude
```

tmux で 3 ペイン以上に分割し、Lead Agent + Worker Agents + Codex CLI を並行運用する。

### Codex CLI Hooks

テンプレートには `.codex/` ディレクトリが含まれ、以下を自動実行する:

| Hook | Timing | Behavior |
|------|--------|----------|
| `SessionStart` | Codex CLI 起動時 | プロジェクトコンテキスト (git branch, CLAUDE.md) をモデルに注入 |
| `Stop` | セッション終了時 | サウンド通知 (Terminal 1 で作業中に終了を検知) |

**サウンドの追加** (任意): `.codex/hooks/sounds/{EventName}/{EventName}.wav` にファイルを配置する。macOS では `afplay` で再生される。

**個人設定**: `.codex/hooks/config/hooks-config.local.json` を作成して hook を無効化できる (git-ignored)。

### Step-by-Step Guide

#### Step 1: PLAN (Terminal 1 -- Claude Code)

```
$ claude
> /plan add-user-authentication
```

Claude Code がコードベースを探索し、`plans/add-user-authentication.md` にフェーズ分割された計画を生成する。計画には各フェーズの Test Gate と空の `<!-- Codex Finding -->` コメントが含まれる。

#### Step 2: QA REVIEW (Terminal 2 -- Codex CLI)

```
$ codex
> plans/add-user-authentication.md の計画をレビューして。
  コードベースと照合し、見落としや問題があれば
  "Codex Finding" として追記して。元のフェーズは書き換えないで。
```

Codex CLI が計画をコードベースと照合し、以下を検証する:
- 既存コードとの整合性
- 見落としているエッジケースや依存関係
- フェーズ間の順序の妥当性

指摘は `<!-- Codex Finding -->` コメントを置き換える形で追記する。**元のフェーズは書き換えない** (非破壊的レビュー)。

> **判断ポイント**: QA REVIEW でアーキテクチャレベルの問題が見つかった場合は Step 1 に戻る。軽微な指摘であればそのまま Step 3 に進む。

#### Step 3: IMPLEMENT (Terminal 1 -- Claude Code, new session)

```
$ claude                  ← 新セッションで起動 (コンテキストをクリーンに保つ)
> plans/add-user-authentication.md を phase-by-phase で実装して
```

**重要**: PLAN セッションとは別のセッションで起動する。計画作成時のコンテキスト (探索結果、議論の経緯) が実装時のコンテキストを汚染するのを防ぐ。

各フェーズ完了後に `/check` を実行し、Test Gate を通過してから次のフェーズに進む。

#### Step 4: VERIFY (Terminal 2 -- Codex CLI)

```
$ codex
> plans/add-user-authentication.md の計画と実装を照合して。
  未実装のフェーズや計画との乖離があれば指摘して。
```

Codex CLI が実装結果を計画と照合し、以下を検証する:
- 全フェーズが実装されているか
- 計画からの逸脱がないか
- テストが十分か

問題がなければ Terminal 1 でコミットする。

### When to Use Each Pattern

| Scale | Recommended Pattern | Steps |
|-------|-------------------|-------|
| New feature / architecture change | Full 4-step cycle | 1 → 2 → 3 → 4 |
| Medium change | Plan + implement only | 1 → 3 |
| Bug fix / small change | Direct implementation | (none) |
| Pre-PR final check | Codex verify only | 4 |

### Session Management Tips

- **PLAN と IMPLEMENT は別セッション** -- コンテキスト汚染を防ぐ最も重要なルール
- **`/compact`** -- コンテキスト使用量が 50% を超えたら実行を検討
- **1 フェーズ = 1 セッション** -- 大きな機能では各フェーズを別セッションにすることも検討
- **Codex CLI は同一セッションで OK** -- Step 2 と Step 4 は同じ Codex セッション内で連続実行できる

### Quick Reference

```
/pixi-env --> /plan --> codex review --> implement --> /check --> codex verify --> /ai-code-changes --> /git-commits
  環境構築     計画作成   QA レビュー     実装        品質検査    最終検証         セルフレビュー       コミット
            (Terminal 1)  (Terminal 2)  (Terminal 1)           (Terminal 2)     (Terminal 1)
```

## Extending

- **Add agents**: Create `.md` files in `.claude/agents/` with YAML frontmatter (`name`, `description`, `model`, `tools`)
- **Add skills**: Create `SKILL.md` in `.claude/skills/<skill-name>/` with YAML frontmatter (`name`, `description`, `user-invocable: true`)
- **Add rules**: Create `.md` files in `.claude/rules/` with plain markdown instructions
- **Add hooks**: Create scripts in `.claude/hooks/` and register them in `.claude/settings.json` under `hooks`
- **Add MCP servers**: Create `.mcp.json` in the project root

## License

[MIT](LICENSE)
