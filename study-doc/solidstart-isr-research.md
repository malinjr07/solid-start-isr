# SolidStart ISR Research Summary

## Issue Overview

**GitHub Issue:** [solidjs/solid-start#443 - Feature Request - Incremental Static Regeneration (ISR)](https://github.com/solidjs/solid-start/issues/443)  
**Created:** November 15, 2022 by @jdgamble555  
**Status:** ✅ Closed as Completed (December 18, 2023)  
**Labels:** `adapter`, `enhancement`

---

## Original Feature Request

### Problem Statement:

- No official way to handle `cache-control` headers in routes for automatic refresh at different intervals
- Confusion in the community about what ISR actually means
- Need for caching API endpoints to save money on database reads and improve page load times
- Ability to trigger revalidation when database updates

### Why ISR Matters:

1. **Cost Savings:** Reduced database reads through caching
2. **Performance:** Faster page loads with static content
3. **Flexibility:** Content updates without full redeployment
4. **Developer Experience:** Simple API for complex caching logic

---

## Key Discussion Points from Issue

### Community Insights:

1. **Comparison with Other Frameworks:**

   - Next.js has mature ISR implementation
   - Nuxt 3 added ISG (Incremental Static Generation)
   - Rich Harris confirmed SvelteKit ISR was priority post-V1

2. **Remix Perspective (Counter-argument):**

   - Remix claims ISR is "vendor lock-in" for Vercel
   - Issue author counters: "All hosting providers for NextJS offer this"
   - Not actually Vercel-specific, but platform-dependent

3. **Implementation Requirements:**
   - Handle inside solid-start core
   - Adapters implement platform-specific caching
   - Support for revalidation triggers

---

## Solution Implemented ✅

### SolidStart Now Supports ISR via Nitro's routeRules

SolidStart leverages **Nitro** (the server engine from Nuxt) which provides built-in ISR support through `routeRules` configuration.

### Configuration in `app.config.ts`:

```typescript
import { defineConfig } from '@solidjs/start/config';

export default defineConfig({
  server: {
    routeRules: {
      // ISR with 60 second revalidation
      '/blog/**': {
        isr: {
          expiration: 60,
        },
      },
      // ISR with 1 hour revalidation
      '/products/**': {
        isr: {
          expiration: 3600,
        },
      },
      // Static prerendering (no revalidation)
      '/about': {
        prerender: true,
      },
    },
  },
});
```

### Key Features:

1. **Per-Route Configuration:**

   - Define different caching strategies per route pattern
   - Support for wildcards (`/blog/**`, `/products/:id`)

2. **ISR Options:**

   ```typescript
   isr: {
     expiration: 60,  // Seconds until revalidation
   }
   ```

3. **Dynamic Routes Support:**
   - Works with dynamic paths like `/product/:id`
   - Pages cached and regenerated even if paths unknown at build time

---

## Alternative Approaches (Pre-Solution)

Before official support, these workarounds were discussed:

### 1. Manual Cache Headers:

```typescript
// In a route handler
export function GET() {
  return new Response(data, {
    headers: {
      'Cache-Control': 's-maxage=60, stale-while-revalidate=600',
    },
  });
}
```

### 2. Platform-Specific Headers:

```typescript
// Vercel-specific
headers: {
  'Cache-Control': 's-maxage=31536000',
  'CDN-Cache-Control': 'max-age=60'
}
```

---

## Prerendering vs ISR in SolidStart

### Static Prerendering (SSG):

```typescript
// app.config.ts
export default defineConfig({
  server: {
    prerender: {
      routes: ['/', '/about', '/blog'],
      crawlLinks: true,
    },
  },
});
```

- Pages generated at **build time**
- Never regenerated until next build

### ISR (Incremental Static Regeneration):

```typescript
// app.config.ts
export default defineConfig({
  server: {
    routeRules: {
      '/blog/**': {
        isr: { expiration: 60 },
      },
    },
  },
});
```

- Pages generated on **first request**
- Regenerated in background after expiration
- Stale content served while regenerating

---

## Technical Architecture

### Nitro-Powered Implementation:

```
┌─────────────────────────────────────────────────────────────┐
│                      SolidStart App                          │
├─────────────────────────────────────────────────────────────┤
│                         Vinxi                                │
│                  (Framework Bundler)                         │
├─────────────────────────────────────────────────────────────┤
│                         Nitro                                │
│               (Universal Server Engine)                      │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                    routeRules                            ││
│  │  - isr: { expiration }                                  ││
│  │  - prerender: true                                      ││
│  │  - cache: { ... }                                       ││
│  │  - headers: { ... }                                     ││
│  └─────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────┤
│                     Deployment Adapters                      │
│    Vercel │ Netlify │ Cloudflare │ Node.js │ etc.          │
└─────────────────────────────────────────────────────────────┘
```

---

## Related Resources

### SolidJS/SolidStart:

- [SolidStart Documentation](https://start.solidjs.com/)
- [Nitro routeRules Docs](https://nitro.unjs.io/config#routerules)
- [GitHub Discussion #1298 - ISR Cache System](https://github.com/solidjs/solid/discussions/1298)

### Cache-Control References:

- [MDN Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control)
- [Vercel Edge Network Caching](https://vercel.com/docs/concepts/edge-network/caching)
- [Netlify Headers](https://docs.netlify.com/routing/headers/)
- [Firebase Hosting Cache](https://firebase.google.com/docs/hosting/manage-cache)

---

## Key Takeaways

1. **ISR is Now Available:** SolidStart fully supports ISR through Nitro's `routeRules`
2. **Simple Configuration:** Define ISR per route in `app.config.ts`
3. **Platform Agnostic:** Works across different hosting providers via adapters
4. **Nitro Foundation:** Leverages battle-tested Nitro server engine (also powers Nuxt 3)
5. **Dynamic Route Support:** ISR works with dynamic parameters, not just static paths

