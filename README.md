# phase-workflow

A reusable two-phase (vibe → mature) development workflow for Claude Code projects.

**Vibe phase** is for exploration: build features, try approaches, discard things. Tests are blocked, dead code is tolerated, and the focus is on making things work.

**Mature phase** is for stability: architecture is settled, so add tests, remove dead code, and enforce strict quality gates.

## What's included

### Scaffold (generic, copied verbatim)

| File | Purpose |
|------|---------|
| `.claude/phase` | Stores current phase (`vibe` or `mature`) |
| `.claude/settings.json` | Claude Code hooks: blocks test creation in vibe, detects maturation signals (regressions, reverts) |
| `.claude/skills/vibe/SKILL.md` | `/vibe` command — switch to exploration mode |
| `.claude/skills/phase/SKILL.md` | `/phase` command — show current phase and constraints |
| `.husky/pre-commit` | Phase-aware pre-commit: types in vibe; lint + types + dead code + security + tests in mature |
| `.husky/commit-msg` | Conventional commits enforcement (mature only) |
| `.lintstagedrc.json` | ESLint + Prettier on staged files |
| `commitlint.config.js` | Conventional commit message rules |

### Templates (adapted per project)

| File | Purpose |
|------|---------|
| `mature-SKILL.md` | `/mature` command — fill in `{{CORE_MODULES}}` with project-specific files |
| `knip.json` | Dead code detection — fill in `{{ENTRY_POINTS}}` and `{{PROJECT_GLOBS}}` |
| `eslint.config.js` | Reference ESLint config with `strictTypeChecked` for `src/` + `api/` |
| `devDependencies.json` | Dev dependencies to merge into `package.json` |
| `scripts.json` | npm scripts to merge into `package.json` |

## Usage

From a project directory, tell Claude Code:

> Integrate the phase workflow from `../phase-workflow`

The bootstrap skill ([SKILL.md](SKILL.md)) handles:

1. Copying scaffold files into the project
2. Detecting entry points, source directories, and core modules
3. Generating `knip.json`, `eslint.config.js`, and the `/mature` skill
4. Merging dev dependencies and scripts into `package.json`
5. Generating `CLAUDE.md` with architecture docs and TypeScript guidance
6. Running `npm install`

The project starts in **vibe** phase.

## Phase behavior

### Vibe

- TypeScript type checking on every commit (catches actually broken code)
- ESLint is **skipped** on commit — fixing lint errors without tests risks regressions
- Dead code detection runs but warns only
- Test file creation is **blocked** by Claude Code hook
- No conventional commit enforcement

### Mature

- ESLint + TypeScript type checking on every commit
- Dead code detection **blocks** commits
- Security scanning (semgrep) **blocks** commits
- Tests must pass to commit
- Conventional commit messages enforced
- Test file creation is **allowed**

### Maturation signals

During vibe phase, Claude Code automatically detects signs that you might be ready to mature:

- **Regression language**: mentions of "broke", "was working", "revert", etc.
- **Multiple reverts**: two or more `git revert`/`git restore`/`git reset` commands

These surface a gentle suggestion to run `/mature`.

## Assumptions

- TypeScript + React + Vite frontend in `src/`
- Node.js/Vercel serverless backend in `api/`
- Separate tsconfig files per source directory (`tsconfig.app.json`, `tsconfig.api.json`)
- `@mathonsunday/mutation-test-gen` available at `../mutation-test-gen`

Adjust the templates if your project structure differs.
