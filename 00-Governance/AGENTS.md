# AGENTS.md

This file is guidance for coding agents working in `cowork-desk`.
It summarizes the current project architecture, commands, and coding conventions.

## Project Snapshot

- Stack: Electron + React + Ant Design + TypeScript + electron-vite.
- Runtime split:
  - Main process: `src/main/**`
  - Preload bridge: `src/preload/**`
  - Renderer UI: `src/renderer/**`
  - Shared contracts/types: `src/shared/**`
- Build output: `out/**` (generated; do not hand-edit).

## Source of Truth for Tooling

- `package.json` scripts are authoritative for build/lint/typecheck commands.
- TS config:
  - Base strictness: `tsconfig.json`
  - Main/preload/shared: `tsconfig.node.json`
  - Renderer/shared: `tsconfig.web.json`
- Vite/Electron aliases: `electron.vite.config.ts`.

## Cursor / Copilot Rules Status

No repository-level Cursor or Copilot instruction files were found at the time of writing:

- `.cursorrules`: not found
- `.cursor/rules/**`: not found
- `.github/copilot-instructions.md`: not found

If these files are added later, update this AGENTS.md and treat those files as higher-priority repo instructions.

## Install / Run Commands

Use npm (lockfile is present).

```bash
npm install
npm run dev
```

`npm run dev` launches:

- renderer dev server (usually `http://localhost:5173`)
- Electron app process

## Build / Lint / Typecheck Commands

```bash
npm run build
npm run lint
npm run typecheck
npm run typecheck:node
npm run typecheck:web
```

Platform package builds:

```bash
npm run build:unpack
npm run build:mac
npm run build:win
npm run build:linux
```

## Test Commands (Current State)

There is currently **no test runner configured** in `package.json` (no `test` script).

That means:

- full test suite command: not available yet
- single test command: not available yet

Until tests are added, validate changes with:

```bash
npm run typecheck
npm run build
```

If/when a runner is introduced (Vitest/Jest/Playwright), add explicit examples here, including a **single test** invocation.

## Single-Test Guidance (When Tests Exist)

Recommended future script shape:

- `npm test` for full suite
- `npm test -- path/to/spec` for one file
- `npm test -- -t "test name"` for one test case

Do not assume these commands work now unless corresponding scripts are added.

## Code Organization Rules

- Keep process boundaries clear:
  - Node/Electron-only code in `src/main` and `src/preload`
  - Browser/UI code in `src/renderer`
  - shared types/constants only in `src/shared`
- Never import renderer modules into main/preload.
- Use IPC channel constants from `src/shared/ipc-channels.ts`.

## Import Conventions

- Prefer absolute aliases where configured:
  - `@shared/*` for shared contracts
  - `@renderer/*` for renderer-only aliases
- Group imports in this order:
  1. external packages
  2. shared/internal aliases
  3. relative imports
- Keep imports type-safe with `import type` where appropriate.

## TypeScript & Types

- Project runs with `strict: true`; keep code strict-clean.
- Avoid `any`; use explicit interfaces/unions and narrowing.
- Put cross-process payload contracts in `src/shared/types.ts`.
- Add optional fields only when truly optional on wire/API boundaries.
- Prefer `unknown` over `any` for untrusted payloads, then refine.

## Naming Conventions

- Components: PascalCase (`SettingsView`, `MainLayout`).
- Hooks/functions/variables: camelCase.
- Constants: UPPER_SNAKE_CASE for global constants.
- IPC channel keys: SCREAMING_SNAKE on `IPC` object, kebab/colon values.
- File names:
  - React component entry files often use `index.tsx`
  - stores use `*Store.ts`

## Formatting & Style

- Follow existing style in repository (2-space indent, semicolon-free TS style).
- Keep functions small and focused.
- Prefer early returns for guard conditions.
- Avoid introducing comments unless behavior is non-obvious.

## State Management (Zustand)

- Keep store selectors stable; avoid returning fresh arrays/objects in selectors.
- When handling stream updates, treat payloads as potentially full snapshots.
- Ensure immutable updates to avoid stale renders.

## IPC & Gateway Practices

- Register IPC handlers centrally in `src/main/ipc.ts`.
- Keep preload surface minimal and explicit (`contextBridge`).
- Never expose raw Node APIs directly to renderer.
- For WebSocket/Gateway events, normalize payloads before renderer consumption.
- Handle reconnect/disconnect transitions explicitly and clear pending requests on disconnect.

## Error Handling Guidelines

- Fail user actions with actionable messages (not silent catches).
- In main process, include enough context in thrown errors for renderer display.
- In renderer, avoid crashing UI on expected transient failures (network/auth/reconnect).
- Distinguish transport errors (not connected) from model/runtime errors.

## Security & Secrets

- Do not hardcode tokens/API keys in source files.
- Keep secrets in user settings/runtime config, not committed code.
- Treat files under user home config paths as sensitive.

## File Editing Boundaries

- Do not edit generated output:
  - `out/**`
  - `node_modules/**`
- Apply source changes under `src/**` and config files at repo root.

## Pre-PR/Pre-Commit Checklist for Agents

Run at minimum:

```bash
npm run typecheck
npm run build
```

If lint is configured/working in the environment, also run:

```bash
npm run lint
```

Then verify:

- Electron app launches (`npm run dev`)
- gateway connection status transitions are sane
- chat streaming renders without duplication/regression

## Updating This File

Update AGENTS.md when any of the following change:

- scripts in `package.json`
- testing framework and single-test command
- lint/format config
- IPC contract patterns
- Cursor/Copilot instruction files
