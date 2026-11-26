---
name: react-data
description: Use when implementing or reviewing the data layer HTTP/SDK clients and interceptors, React Query queries/mutations, query keys, invalidation, optimistic updates, retries, HTTP error handling, or mock data (MSW/Firebase emulators). Load this before adding an API call, a query/mutation or a mock.
---

# React Data

Owns **the data layer**: transport clients (HTTP/SDK), server state (React Query), HTTP/SDK error handling and mock data (MSW / SDK emulators). Load this before adding an API call, a query/mutation or a mock.

## When to use

- Creating or reviewing a transport client or interceptor.
- Adding a query, mutation, query key, invalidation or optimistic update.
- Handling HTTP/SDK errors or defining an error type.
- Setting up or reviewing mock data (MSW, Firebase emulators).
- Deciding where data-related files live.

For feature structure, read `react-feature`. For layers and imports, read `react-architecture`. For shared folder, read `react-shared`. For library stack, read `react-stack`. For forms, read `react-forms`. For routing, read `react-routing`. For testing, read `react-testing`. For effects, read `react-effects`.

## Transport clients

- Location of transport clients: `src/shared/apis/<client>/client.ts` (e.g. `shared/apis/axios/client.ts`, `shared/apis/firebase/client.ts`).
- One instance per client. Create the folder only when the client is used (no empty folders).
- Configuration (base URL, timeout, `withCredentials`) comes from `shared/config/env.ts`. Never read `import.meta.env` elsewhere.
- `shared/apis/` holds **two different things**:
  - **Transport clients** (`<client>/client.ts`): shared infrastructure, domain-agnostic (Axios instance, Firebase client). Always shared.
  - **Promoted data modules** (`services/`, `queries/`, `mutations/`): only when a module is promoted to shared (used by 2+ features or app+1 feature, domain-agnostic, with approval). See `react-shared` promotion gate.
- Feature data access (`services/`, `queries/`, `mutations/`) **stays in the feature** (`features/<f>/apis/`). See `react-feature`. Do not move a feature's data access to `shared/apis/` without promotion approval.

### Axios

- Instance in `shared/apis/axios/client.ts`.
- Set default headers (e.g. `Content-Type: application/json`), timeout and `withCredentials` as needed.
- `withCredentials: true` **when the project uses cookie-based sessions**.
- **Token injection is opt-in and requires project approval.** When enabled, inject the bearer token from the session store / `sessionStorage` in the request interceptor.
- Request interceptor (optional): correlation ID / request id, dev logging.
- Response (success): return `response.data` (be consistent across the app).
- Response (error): normalize to `ApiError` and reject.
- **401 handling:** attempt a token refresh once; on failure, clear the session and redirect to login. Guard against infinite refresh loops.
- Support `AbortController` / `signal` in services to cancel in-flight requests.
- File uploads: multipart/form-data with `onUploadProgress`.

### Other SDKs (Firebase, etc.)

- Same folder convention: `shared/apis/<sdk>/client.ts` (and `config.ts`).
- Auth and client setup follow the SDK's own rules.
- Mocks do **not** go through MSW; use the SDK's emulator or in-memory fakes (see Mock data).

### Normalized error

```ts
// src/shared/types/apiError.type.ts (or interfaces/)
export interface ApiError {
  status: number;        // HTTP status (0 for network/CORS/SDK failure)
  message: string;       // user-facing or fallback message
  code?: string;         // backend/SDK error code if provided
  details?: unknown;     // validation details, etc.
}
```

- Map Axios/SDK errors to `ApiError` in the client/interceptor.
- Feature code and UI consume `ApiError`, never `AxiosError` or SDK errors.
- Use `ApiError.status` / `code` to branch (401, 403, 404, 422, 5xx).

## React Query

### QueryClient defaults

- Configure in `src/app` (composition root) and wrap the app in `QueryClientProvider`.
- `staleTime`: sensible default (e.g. 30s–1min); override per query when needed.
- `retry`: 1–2 for queries, 0 for mutations.
- Retry with exponential backoff; do not retry 4xx (except 429, honoring `Retry-After`).
- `refetchOnWindowFocus`: decide per app (default true is fine for most).
- React Query Devtools only in dev (optional, requires approval).

### Query keys

- Co-located in `features/<f>/apis/queries/` next to each `useQuery`.
- Use a factory:

```ts
export const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (filters: UserFilters) => [...userKeys.lists(), filters] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id: string) => [...userKeys.details(), id] as const,
};
```

### Queries

- Wrap `useQuery` in a feature hook in `apis/queries/` (e.g. `useUsers`, `useUser(id)`).
- Components consume the hook, never `useQuery` directly with raw keys.
- Use `placeholderData: keepPreviousData` for paginated lists; `useInfiniteQuery` for infinite scroll.
- Prefetching is optional (`queryClient.prefetchQuery` on hover/route).

### Mutations

- Wrap `useMutation` in `apis/mutations/` (e.g. `useCreateUser`, `useUpdateUser`, `useDeleteUser`).
- `onSuccess`: invalidate affected query keys (data concern, stays in the hook).
- The hook **returns the error**; the caller (component) decides toast and `setError` (see `react-forms`). Mutation hooks do not trigger toasts or form field errors by default.
- Optional: a reusable `mapErrorToFields(error)` in `utils/` for field-error mapping reused across forms.
- Prevent double submit at the UI (disable button while pending).
- Navigation lives in the component.

### Invalidation

- Invalidate by query key in `onSuccess` of the mutation.
- Prefer targeted keys (`userKeys.detail(id)`) over broad (`userKeys.all`) when possible.
- Clear the whole cache on logout (`queryClient.clear()`) to avoid data leakage between sessions.

### Optimistic updates

- Use `onMutate` to update the cache, `onError` to rollback, `onSettled` to invalidate/refetch.
- Always implement rollback; never leave the cache inconsistent on failure.

### Error boundaries

- Data errors are surfaced via error boundaries per feature/route (aligned with `react-routing`).
- Queries can use `throwOnError` for boundary handling; otherwise handle inline.

### Type safety

- Services return typed data; hooks expose typed queries/mutations.
- Use a generic `ApiResponse<T>` (or unwrap in the service) and keep it consistent.

## Errors in the UI

- Unexpected errors → error boundary.
- Validation errors → inline, next to the field.
- Mutation errors → the component shows toast and/or `setError` (see `react-forms`); the hook returns the error.
- Loading states → skeletons / spinners; use `isPending` / `isLoading`.

## Mock data

### HTTP mocks (MSW)

- Location: `mocks/` at the repo root, **sibling of `src/` and `tests/`**.
- Structure:

```text
mocks/
  browser.ts          # setupWorker for dev
  server.ts           # setupServer for tests
  handlers/
    index.ts          # aggregates all handlers
    users.handlers.ts # per feature/domain
  factories/
    user.factory.ts   # data factories
```

- Handlers mirror the real API: same paths, methods, status codes and response shapes.
- One handler file per feature/domain; aggregate in `handlers/index.ts`.
- Use `delay()` with `VITE_MOCK_DELAY` to simulate latency.
- Include error handlers (4xx/5xx) to test error paths.
- Use factories for realistic, varied data; reset handlers between tests (`server.resetHandlers()`).
- Factory location: `mocks/factories/` (root) is the single source of mock data, shared by MSW handlers (dev) and tests. `tests/factories/` (or `tests/fixtures/`) is reserved for test-only data that is not an API mock (component props, store fixtures). Do not duplicate a mock factory in both places.
- Keep handlers stateful when needed (in-memory store) for write-then-read flows.
- Toggle via `VITE_USE_MOCKS` (read only in `shared/config/env.ts`).
- In dev, start the worker conditionally when the flag is on.
- In tests, use `server.ts` with `beforeAll`/`afterEach`/`afterAll` (see `react-testing`). Tests import handlers/factories from the root `mocks/`.
- Mocks must not diverge silently from the real API; update handlers when the contract changes.

### SDK mocks (Firebase, etc.)

- Do **not** use MSW for SDKs; use the SDK's emulator or in-memory fakes.
- Firebase example:
  - Services: Auth + Firestore (extensible to Storage/Functions).
  - Use **Firebase Emulator Suite** for dev and tests.
  - Location: `shared/apis/firebase/` (client, config, emulator wiring).
  - Toggle: `VITE_USE_FIREBASE_EMULATOR` (read in `shared/config/env.ts`).
  - Wire `connectAuthEmulator`, `connectFirestoreEmulator`, etc. only when the flag is on.
  - Test security rules with `firebase emulators:exec` (or rules tests).
  - Seed data via scripts in `mocks/firebase/` or `scripts/`; run before E2E or in test setup.
  - Unit tests may use SDK fakes for speed; E2E uses the emulator.
  - Never connect to the emulator in production (gate by env flag).
- Generic pattern for any non-HTTP SDK: client in `shared/apis/<sdk>/`, mock via emulator/fake, toggle via env flag, seed scripts, never in prod.

## Environment variables

Read and validated **only** in `shared/config/env.ts`. The list below is an **example**; each client declares its own required variables.

| Variable | Purpose | Required |
| --- | --- | --- |
| `VITE_API_BASE_URL` | Base URL of the HTTP client | No (optional) |
| `VITE_API_TIMEOUT` | Request timeout (ms) | No (default 15000) |
| `VITE_USE_MOCKS` | Enable MSW (dev + tests) | No (default false) |
| `VITE_MOCK_DELAY` | Artificial mock delay (ms) | No (default 0) |
| `VITE_USE_FIREBASE_EMULATOR` | Enable Firebase emulators | No (default false) |
| `VITE_FIREBASE_API_KEY` | Firebase config | Only if using Firebase |
| `VITE_FIREBASE_AUTH_DOMAIN` | Firebase config | Only if using Firebase |
| `VITE_FIREBASE_PROJECT_ID` | Firebase config | Only if using Firebase |
| `VITE_FIREBASE_STORAGE_BUCKET` | Firebase config | Only if using Firebase |
| `VITE_FIREBASE_MESSAGING_SENDER_ID` | Firebase config | Only if using Firebase |
| `VITE_FIREBASE_APP_ID` | Firebase config | Only if using Firebase |

## Anti-patterns (do not)

- Reading `import.meta.env` outside `shared/config/env.ts`.
- Calling a transport client directly in components (use services + hooks).
- Using `AxiosError` / SDK errors in UI (use `ApiError`).
- Hardcoding base URLs or endpoints.
- Query keys as magic strings scattered in components.
- Mutations without invalidation.
- Optimistic updates without rollback.
- Mocks that drift from the real API contract.
- Duplicating a mock factory in both `mocks/factories/` and `tests/factories/`.
- Assuming MSW covers non-HTTP SDKs.
- Connecting to an emulator in production.
- Empty `mocks/` or `shared/apis/<client>/` folders.

## Checklist

- [ ] Transport client in `shared/apis/<client>/client.ts`; config from env.
- [ ] `withCredentials` set when using cookie sessions.
- [ ] Token injection only if approved.
- [ ] Interceptors normalize errors to `ApiError`; 401 refresh guarded.
- [ ] React Query defaults set (staleTime, retries, backoff).
- [ ] Query keys co-located with `useQuery` and exported as a keys object.
- [ ] Mutations invalidate affected keys; errors returned to caller (toast/setError in component).
- [ ] Optimistic updates include rollback.
- [ ] Cache cleared on logout.
- [ ] Error boundaries per feature/route where appropriate.
- [ ] HTTP mocks in `mocks/` (root) with handlers + factories; error handlers included.
- [ ] SDK mocks use emulator/fakes, never in prod.
- [ ] Env vars documented as example; each client declares its own.
- [ ] Lint and typecheck pass.

## References

| Situation | Read |
| --- | --- |
| Feature structure and isolation | `react-feature` |
| Layer rules and imports | `react-architecture` |
| Shared folder, promotion and shadcn location | `react-shared` |
| Library stack and versions | `react-stack` |
| Testing strategy, structure and coverage | `react-testing` |
| Effect policy (useEffect, etc.) | `react-effects` |
| Forms (RHF + Zod) | `react-forms` |
| Routing (paths, guards, params) | `react-routing` |
| Accessibility (semantics, focus, contrast, etc.) | `react-a11y` |
| Performance (splitting, memo, virtualization, profiling) | `react-performance` |
| Conventions (commits, PRs, branching, review) | `react-conventions` |