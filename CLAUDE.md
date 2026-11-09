# Genuivo Types

> **Git policy — never auto-commit or auto-push.** Leave your work in the working tree.
> Run `git commit`, `git push`, `gh pr create`, or `scripts/push_all.sh` **only when the user
> explicitly asks in that turn**. Approval for an earlier change does not carry forward, and
> finishing a task is not permission to commit it.

Shared TypeScript type definitions for the Genuivo API ecosystem. Foundation
package: domain models, request/response shapes, and small runtime response
helpers that every other `genuivo_*` project depends on.

**npm**: `@sudobility/genuivo_types` (GitHub Packages, BUSL-1.1)

## Tech Stack

- **Language**: TypeScript (strict mode), `"type": "module"`
- **Package manager**: Bun (do not use npm/yarn/pnpm to install)
- **Build**: `tsc` — **ESM-only** output (no CJS, despite older notes)
- **Test**: Vitest

## Project Structure

```
src/
├── index.ts        # All types, interfaces, response helpers, type guards (single barrel)
└── index.test.ts   # Vitest tests (81 tests)
```

## Commands

```bash
bun install            # Install deps (needs ~/.npmrc auth for @sudobility scope)
bun run build          # Compile ESM to dist/ (tsc -p tsconfig.esm.json)
bun run dev            # tsc --watch
bun run typecheck      # tsc --noEmit
bun run lint           # ESLint over src
bun run test:unit      # Run Vitest once  (NOT `bun test` — that is Bun's own runner)
bun run test:watch     # Vitest watch
bun run verify         # typecheck + lint + test:unit + build (run before publishing)
bun run prepublishOnly # clean + verify (runs on publish)
```

## Exports (src/index.ts)

- **Domain**: `User`, `History`, `ISODateString` (branded string)
- **Request/response**: `HistoryCreateRequest`, `HistoryUpdateRequest`,
  `HistoryTotalResponse`, `ChatRequest`, `ChatResponse`, `ApiInfoResponse`,
  `HealthResponse`
- **Runtime helpers**: `successResponse<T>(data)`, `errorResponse(error)` — both
  build a `BaseResponse` envelope with `{ success, ..., timestamp }`
- **Type guards**: `isSuccessResponse<T>(res)`, `isErrorResponse(res)`
- **Re-exports**: `ApiResponse`, `BaseResponse`, `NetworkClient`, `Optional`
  (from `@sudobility/types`); `IRenderable` (from `@sudobility/genui_types`)

## Peer Dependencies

- `@sudobility/types` — base infrastructure types (`BaseResponse`, `NetworkClient`, …)
- `@sudobility/genui_types` — `IRenderable` for chat/GenUI types

## Architecture

```
@sudobility/genuivo_types (this package)
    ^
genuivo_api  genuivo_client  genuivo_lib  genuivo_app  genuivo_app_rn
```

Foundation layer — all other Genuivo projects depend on this package.

- **genuivo_api** — Backend API (Hono + PostgreSQL); uses types for validation
- **genuivo_client** — API client SDK (TanStack Query); defines API contracts
- **genuivo_lib** — Business logic (Zustand); imports types via genuivo_client
- **genuivo_app** — Web app (React + Vite); imports transitively
- **genuivo_app_rn** — React Native app; imports transitively

## Conventions

- Pure type definitions; the only runtime logic is the four helpers/guards above.
- All public exports go through the single `src/index.ts` barrel.
- `interface` for object shapes, `type` for unions/aliases/brands.
- Type names track API naming: `History`, `HistoryCreateRequest`, etc.
- Re-export base types from `@sudobility/types` so consumers depend only on this package.
- Nullable DB-backed timestamp fields are `string | null`.

## Gotchas

- **Downstream blast radius**: changes here affect ALL consumers (genuivo_api,
  _client, _lib, _app, _app_rn). Always consider downstream impact.
- **ESM-only**: `build` runs only `tsc -p tsconfig.esm.json`; there is no CJS
  target. `exports` exposes only `import`/`types`.
- **Run `bun run verify` before publishing** to catch type/lint/test/build failures.
- `BaseResponse<T>` is the standard API envelope — all API responses conform to it.
- Keep this a lightweight type-only dependency; add no runtime deps (the response
  helpers/guards are the sole runtime code).
- Two ESLint configs exist (`eslint.config.js` and `eslint.config.mjs`); ESLint
  resolves `eslint.config.js` first, so the `.mjs` is effectively unused.

## Git Workflow

- Do not use feature branches for code changes. Always stay on the current branch.
