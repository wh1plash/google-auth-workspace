---
name: auth-frontend-feature
description: Add or change SPA pages in google-auth-frontend — TanStack Router file routes, api.ts client, React Query, auth guards, Tailwind UI. Use for UI features and API integration.
---

# Frontend Feature

## When to use

- New page or route
- API integration, forms, auth-gated views
- Profile, credentials, password reset UI

## Hard rules

1. All API calls go through `src/lib/api.ts` — do not scatter `fetch` calls.
2. After adding route files, run `npm run routes:generate` (or `npm run build`).
3. Never commit `.env` — use `.env.example` for documented vars.
4. Session token queries must be scoped (e.g. `['me', sessionToken]`) to prevent cross-user cache bleed.

## Procedure

### 1. API method

Add typed function in `src/lib/api.ts`:

```typescript
export const api = {
  // ...
  myFeature: (body: MyBody) => request<MyResponse>('/api/my/path', { method: 'POST', body }),
}
```

Use `authHeaders()` for protected endpoints.

### 2. Route file

Create `src/routes/my-feature.tsx`:

```typescript
import { createFileRoute } from '@tanstack/react-router'

export const Route = createFileRoute('/my-feature')({
  component: MyFeaturePage,
})
```

### 3. Regenerate route tree

```bash
npm run routes:generate
```

### 4. Auth guard (if needed)

```typescript
beforeLoad: () => {
  if (!hasValidSession()) throw redirect({ to: '/login' })
}
```

### 5. React Query

```typescript
const { data } = useQuery({
  queryKey: ['me', sessionToken],
  queryFn: () => api.me(sessionToken),
  enabled: !!sessionToken,
})
```

## Key files

| Purpose | Path |
|---------|------|
| API client | `src/lib/api.ts` |
| Session storage | `src/lib/auth-storage.ts` |
| Routes | `src/routes/*.tsx` |
| Credentials UI | `src/components/CredentialsPanel.tsx` |
| UI primitives | `src/components/ui/` |

## Docker

`VITE_API_BASE_URL` baked at image build. Local dev: `npm run dev` with `.env` containing `VITE_API_BASE_URL=http://localhost:8090`.

## Lint / build

```bash
npm run lint
npm run build
```
