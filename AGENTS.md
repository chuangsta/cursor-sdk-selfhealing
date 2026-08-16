# AGENTS.md

## Cursor Cloud specific instructions

This repo is a single Node.js + TypeScript CLI product: the **ADE Task Debugger**
(`ade`), which uses the Cursor SDK to diagnose failed Snowflake Tasks and open fix
PRs. There are no separate services, no database, and no web frontend — everything
runs as a one-shot CLI. See `README.md` for the product overview and full CLI/demo
reference.

### Environment / tooling
- Node 20+ and pnpm are required; pnpm is pinned via `packageManager` in `package.json`.
  Dependencies are installed with `pnpm install` (handled by the startup update script).
- Standard scripts live in `package.json` (`typecheck`, `build`, `ade`, `ade:dist`, `start`).

### Lint / test / build / run
- **Lint:** there is no ESLint/Prettier config. The lint equivalent is the strict
  TypeScript compile: `pnpm typecheck` (`tsc --noEmit`).
- **Tests:** there is no test framework or test suite in this repo. Do not assume
  `pnpm test` exists.
- **Build:** `pnpm build` (emits `dist/` via `tsc`).
- **Run (dev):** `pnpm ade <args>` runs the CLI from source via `tsx` (no build step).
  `pnpm ade:dist` / `pnpm start` run the compiled `dist/cli.js` instead.

### Non-obvious gotchas
- **No credentials needed for local dev/demo.** Pass `--dry-run` (or set `ADE_DRY_RUN=1`)
  to exercise the full flow using `examples/sample-failure.json`. Live runs require
  `CURSOR_API_KEY`, `ADE_TARGET_REPO`, and `ADE_SNOWFLAKE_MCP_URL`; if any are missing,
  a non-`--dry-run` invocation automatically **falls back to the fixture/dry path**
  instead of erroring (see `debugTask` in `src/orchestrator.ts`).
- Config is read from a real `.env` file at the repo root (loaded by `src/config.ts`);
  copy `.env.example` → `.env` for live runs. Never commit `.env` or real tokens.
- Run observability is written to `.runs/` (JSON + `runs.jsonl`), which is gitignored.
  Inspect with `pnpm ade runs list` and `pnpm ade runs show <localRunId|runId|agentId>`.
- Quick offline smoke test:
  `pnpm ade debug-task ADE_DEMO.OPS.LOAD_DAILY_ORDERS --change-object ADE_DEMO.OPS.ORDERS --change-type rename_column --change-before AMOUNT --change-after ORDER_TOTAL --dry-run`
