# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

oh-pi is a one-click configuration tool for pi-coding-agent (like oh-my-zsh for pi). It provides an interactive TUI for setting up API providers, themes, keybindings, extensions, and agent templates. An optional advanced feature is the **Ant Colony** multi-agent orchestration system built on real ant colony behavior patterns.

**Current version**: 0.1.76 | **License**: MIT | **Node.js**: >=20.0.0

## Commands

```bash
npm run build        # TypeScript compile (tsc) → dist/
npm run dev          # Watch mode (tsc --watch)
npm run test         # Run all tests (vitest run)
npm start            # Run CLI (node dist/bin/oh-pi.js)
```

Run a single test file:
```bash
npx vitest run src/i18n.test.ts
npx vitest run pi-package/extensions/ant-colony/queen.test.ts
```

Tests are co-located with source files (`.test.ts` alongside `.ts`).

## Architecture

### Two-Layer Separation

```
src/                    → Configuration TUI (installer)
  bin/oh-pi.ts            CLI entry point (Windows UTF-8 aware)
  index.ts                Main orchestrator: run() → quickFlow | presetFlow | customFlow
  tui/                    9 interactive screens (welcome → mode → provider → theme → ... → confirm)
  utils/                  detect.ts (env detection), install.ts (apply config), writers.ts (file generators)
  types.ts                OhPConfig, ProviderConfig, etc.
  i18n.ts + locales.ts    Internationalization (zh/en/fr)
  registry.ts             Runtime constants (providers, themes, extensions)

pi-package/             → Distributable resources (installed into ~/.pi/agent/)
  extensions/             8 extensions + ant-colony/ subsystem
  skills/                 11 skills (context7, web-search, design systems, etc.)
  prompts/                10 prompt templates (review, fix, commit, etc.)
  agents/                 5 agent role templates
  themes/                 6 themes
```

The `src/` layer is the installer TUI. The `pi-package/` layer contains resources that get symlinked/copied into the user's `~/.pi/agent/` directory.

### Ant Colony System (pi-package/extensions/ant-colony/)

A bio-inspired multi-agent orchestration system (~2200 lines) mapping real ant colony behaviors:

| Module | Role |
|--------|------|
| `types.ts` | Foundation types: Task, Ant, Pheromone, Caste, Nest (leaf node, 8 dependents) |
| `queen.ts` | Lifecycle orchestrator: scouting → planning → working → reviewing → done |
| `nest.ts` | Shared state via filesystem (.ant-colony/), atomic reads/writes, pheromone management |
| `spawner.ts` | Process management: spawn pi sessions, build prompts, parse output |
| `concurrency.ts` | Adaptive concurrency: cold start → explore → steady state → overload protection |
| `parser.ts` | Output parsing: extract tasks, pheromones, sub-tasks from agent output |
| `prompts.ts` | System prompts per caste (scout/worker/soldier) |
| `deps.ts` | Dependency graph: file-level conflict detection |
| `ui.ts` | Formatting helpers (duration, cost, tokens) |
| `index.ts` | Extension entry: tool/command registration (~600 lines) |

**Caste system**: Scout (haiku, read-only, fast recon) → Worker (sonnet, full tools, execution) → Soldier (sonnet, read-only, quality review)

**Pheromone communication**: JSONL append-only log with 10-minute half-life exponential decay. Six pheromone types for indirect agent coordination.

### Config Flow

`run()` in `src/index.ts` drives three modes:
- **quick**: provider + theme only, sensible defaults for everything else
- **preset**: choose a preset (Starter/Pro/Security/Data), then configure provider
- **custom**: step-by-step selection of all options

All modes end at `confirmApply()` which previews config, backs up existing, and writes to `~/.pi/agent/`.

## Key Conventions

- **Pure ESM** (`"type": "module"` in package.json). All imports use `.js` extensions.
- **TypeScript strict mode** with ES2022 target, Node16 module resolution.
- **Runtime dependencies are minimal**: only `@clack/prompts` and `chalk`.
- **pi integration** declared via `"pi"` field in package.json (extensions, skills, prompts, themes paths).
- Colony state lives in `.ant-colony/{colony-id}/` directories (gitignored).

## Architecture Decisions (DECISIONS.md)

- **D-001**: oh-pi is the primary identity; ant-colony is an optional advanced extension.
- **D-002**: Growth focuses on Chinese developer community first.
- **D-003**: PheromoneStore interface and PiAdapter anti-corruption layer are planned before deeper optimization (see `docs/PHEROMONE-STORE.md` and `docs/PI-ADAPTER.md`).

## Planned Phases (PLAN.md)

| Phase | Focus | Versions |
|-------|-------|----------|
| A (done) | README alignment, UI polish | v0.1.76 |
| B | PheromoneStore abstraction | v0.1.77-78 |
| C | PiAdapter anti-corruption layer | v0.1.79 |
| D | Benchmark suite, community | v0.1.80 |
