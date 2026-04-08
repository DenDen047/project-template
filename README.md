# project-template

Minimal Python project template with [pixi](https://pixi.sh) and [Claude Code](https://claude.ai/code).

## Prerequisites

- [nanokit](https://github.com/DenDen047/nanokit) -- dev environment (pixi, shell, CLI tools)
- [claude-settings](https://github.com/DenDen047/claude-settings) -- Claude Code MCP configuration

## Setup

1. Create a new repo from this template on GitHub, then clone it
   ```bash
   git clone https://github.com/<you>/<your-project>.git
   cd <your-project>
   ```
2. Initialize pixi environment with Claude Code
   ```
   /pixi-env
   ```
3. Install dependencies
   ```bash
   pixi install
   ```

## Usage

```bash
pixi run test      # Run tests
pixi run lint      # Lint check
pixi run format    # Auto-format
```

## Structure

```
├── CLAUDE.md       # Claude Code project instructions
├── .gitignore      # Python + pixi
├── src/            # Source code
└── tests/          # Tests
```

`pixi.toml` is not included -- generate it per project with the `/pixi-env` skill or `pixi init`.

## Workflow

This template assumes the following Claude Code skills from [nanokit](https://github.com/DenDen047/nanokit):

| Phase | Skill | Description |
|-------|-------|-------------|
| Environment setup | `/pixi-env` | Initialize `pixi.toml` with conda/PyPI dependencies |
| Coding | `/ai-code-changes` | Pre-change investigation and self-review |
| Coding | `/code-comments` | Write meaningful comments (Why, not What) |
| Review | `/code-review-etiquette` | Actionable review with blocking/nit distinction |
| Commit | `/git-commits` | One logical change per commit, Problem -> Solution message |
| Documentation | `/readme-writing` | Funnel-structured README updates |

Typical flow:

```
/pixi-env  ->  code  ->  /ai-code-changes  ->  /git-commits
```

## License

[MIT](LICENSE)
