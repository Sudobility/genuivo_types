# @sudobility/genuivo_types

Shared TypeScript type definitions for the Genuivo API ecosystem. This is the
foundation package: it defines the API domain models, request/response shapes,
and a few tiny runtime response helpers that every other `genuivo_*` project
depends on. It ships ESM-only type declarations plus a small amount of runtime
JavaScript (the response helpers and type guards).

- **npm name**: `@sudobility/genuivo_types` (published to GitHub Packages)
- **License**: BUSL-1.1

## Installation

`@sudobility` packages are hosted on GitHub Packages, not the public npm
registry. The committed `.npmrc` routes the `@sudobility` scope to
`https://npm.pkg.github.com`; your own auth token must be present in
`~/.npmrc`.

```bash
bun add @sudobility/genuivo_types
```

Peer dependencies (install alongside):

```bash
bun add @sudobility/types @sudobility/genui_types
```

## Usage

```ts
import {
  History,
  HistoryCreateRequest,
  HistoryUpdateRequest,
  HistoryTotalResponse,
  ChatRequest,
  ChatResponse,
  User,
  successResponse,
  errorResponse,
  isSuccessResponse,
  isErrorResponse,
  type BaseResponse,
  type NetworkClient,
} from "@sudobility/genuivo_types";

// Wrap API responses in the standard BaseResponse envelope
const ok = successResponse({ items: [] });
// { success: true, data: { items: [] }, timestamp: "<ISO 8601>" }

const err = errorResponse("Not found");
// { success: false, error: "Not found", timestamp: "<ISO 8601>" }

// Narrow a response with the type guards
function handle(res: BaseResponse<History[]>) {
  if (isSuccessResponse(res)) {
    return res.data; // typed as History[]
  }
  throw new Error(res.error);
}
```

## Exports

### Domain types

| Type | Description |
|------|-------------|
| `User` | User account (`firebase_uid`, nullable `email`/`display_name`/timestamps) |
| `History` | Core entity (`id`, `user_id`, `datetime`, `value`, nullable timestamps) |
| `ISODateString` | Branded string alias for ISO 8601 datetimes |

### Request / response types

| Type | Description |
|------|-------------|
| `HistoryCreateRequest` | `{ datetime, value }` |
| `HistoryUpdateRequest` | `{ datetime?, value? }` (partial update) |
| `HistoryTotalResponse` | `{ total }` |
| `ChatRequest` | `{ request }` — chat endpoint input |
| `ChatResponse` | `{ output: IRenderable }` — GenUI render tree |
| `ApiInfoResponse` | `{ name, version, status }` — `GET /` |
| `HealthResponse` | `{ status, version }` — `GET /health` |

### Runtime helpers and guards

| Export | Description |
|--------|-------------|
| `successResponse<T>(data)` | Builds `BaseResponse<T>` with `success: true` + timestamp |
| `errorResponse(error)` | Builds `BaseResponse<never>` with `success: false` + timestamp |
| `isSuccessResponse<T>(res)` | Type guard narrowing to the success branch |
| `isErrorResponse(res)` | Type guard narrowing to the error branch |

### Re-exports

- From `@sudobility/types`: `ApiResponse`, `BaseResponse`, `NetworkClient`, `Optional`
- From `@sudobility/genui_types`: `IRenderable`

Consumers only need to depend on this package for these base types.

## Development

Dependencies are managed with **Bun** (do not use npm/yarn/pnpm).

```bash
bun install            # Install dependencies
bun run build          # Compile ESM output to dist/ (tsc -p tsconfig.esm.json)
bun run dev            # tsc --watch
bun run typecheck      # tsc --noEmit
bun run lint           # ESLint over src
bun run lint:fix       # ESLint --fix
bun run format         # Prettier --write
bun run test:unit      # Run Vitest once
bun run test:watch     # Vitest watch mode
bun run test:coverage  # Vitest with coverage
bun run verify         # typecheck + lint + test:unit + build (run before publishing)
```

Note: tests run with **Vitest**, not Bun's built-in test runner — use
`bun run test:unit`, not `bun test`.

## Build output

Build is **ESM-only**. `tsc -p tsconfig.esm.json` emits `dist/index.js`,
`dist/index.d.ts`, and source/declaration maps. The package is `"type":
"module"` and its `exports` map only exposes the `import`/`types` conditions.

## Related packages

All other `genuivo_*` projects depend on this one:

- **genuivo_api** — Backend API server (Hono + PostgreSQL)
- **genuivo_client** — API client SDK with TanStack Query hooks
- **genuivo_lib** — Business logic library with Zustand stores
- **genuivo_app** — React + Vite web application
- **genuivo_app_rn** — React Native mobile app

## Publishing

CI (`.github/workflows/ci-cd.yml`) uses the shared `johnqh/workflows`
reusable workflow and publishes to GitHub Packages on push to `main` /
`develop`. `prepublishOnly` runs `clean` + `verify`.

## License

BUSL-1.1
