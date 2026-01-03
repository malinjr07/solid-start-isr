# SvelteKit ISR Research Summary

## Issue Overview

**GitHub Issue:** [sveltejs/kit#661 - Add incremental static regeneration](https://github.com/sveltejs/kit/issues/661)  
**Created:** March 25, 2021 by @Nick-Mazuk  
**Status:** Open (labeled as `feature / enhancement`, `size:large`, milestone: `soon`)  
**Labels:** `feature / enhancement`, `size:large`

---

## What is ISR (Incremental Static Regeneration)?

ISR is a Next.js feature that combines benefits of:

- **Static Site Generation (SSG):** Fast page loads with pre-rendered HTML
- **Server-Side Rendering (SSR):** Fresh content without full rebuilds

### Core Features:

1. Pages can be dynamically generated on first request
2. Resulting page is cached for X seconds/minutes
3. After X time, page is regenerated in the background on next request
4. Optional: On-demand cache invalidation via API

---

## Original Proposed Solution

Extend SvelteKit's prerender API with a `revalidate` parameter:

```svelte
<script context="module">
  export const prerender = true;
  export const revalidate = 900; // 900 seconds, or 15 minutes
</script>
```

---

## Community Discussion & Insights

### Key Points Raised:

1. **Platform-Specific Implementation Required**

   - Cache invalidation for ISR is often platform-specific
   - Implementation depends heavily on deployment platform (Vercel, Netlify, Cloudflare, etc.)
   - Each platform has different cache-control mechanisms

2. **Current Workarounds:**

   - Use `stale-while-revalidate` cache-control headers manually
   - Platform-specific adapters (e.g., Vercel adapter already supports ISR)
   - Using fetch with `revalidate` option in server load functions

3. **Vercel Adapter Support:**

   - Vercel explicitly supports ISR for SvelteKit applications
   - Can use `revalidate` option within fetch requests in `+page.server.js`:

   ```javascript
   // +page.server.js
   export async function load({ fetch }) {
     const res = await fetch('/api/data', {
       next: { revalidate: 60 }, // Revalidate every 60 seconds
     });
     return { data: await res.json() };
   }
   ```

4. **Rich Harris Statement:**
   - Mentioned that ISR would be a priority feature after SvelteKit V1 release

### Related Issues:

- **#2369** - Incremental builds support for static adapter
- **#3199** - User-updatable static content without full rebuilds

---

## Current State (as of Jan 2024)

### What's Available:

1. **Vercel-Specific ISR:**

   - SvelteKit apps deployed on Vercel can use ISR via:
     - Functions config in `svelte.config.js`
     - Fetch revalidation headers
     - Edge Functions with caching

2. **Manual Cache-Control:**

   - Set cache headers in server hooks:

   ```javascript
   // hooks.server.js
   export async function handle({ event, resolve }) {
     const response = await resolve(event);
     response.headers.set(
       'Cache-Control',
       's-maxage=60, stale-while-revalidate=600'
     );
     return response;
   }
   ```

3. **Adapter-Specific Solutions:**
   - `@sveltejs/adapter-vercel` - Full ISR support
   - `@sveltejs/adapter-netlify` - Partial support via CDN caching
   - `adapter-cloudflare` - Workers KV caching options

### What's Missing (Native SvelteKit):

- Framework-level `revalidate` export option
- On-demand revalidation API
- Platform-agnostic ISR configuration
- Built-in cache invalidation triggers

---

## Proposed Implementation Approaches

### Approach 1: Framework-Level Page Config

```javascript
// +page.server.js
export const config = {
  isr: {
    expiration: 60, // seconds
  },
};
```

### Approach 2: Load Function Return Option

```javascript
export async function load({ fetch }) {
  return {
    data: await fetchData(),
    __revalidate: 60,
  };
}
```

### Approach 3: Svelte Config Integration

```javascript
// svelte.config.js
export default {
  kit: {
    isr: {
      routes: {
        '/blog/*': { revalidate: 3600 },
        '/products/*': { revalidate: 60 },
      },
    },
  },
};
```

---

## Key Challenges Identified

1. **Platform Abstraction:** How to abstract ISR across different hosting platforms?
2. **Cache Storage:** Where to store cached pages (CDN, Redis, file system)?
3. **Invalidation Mechanism:** How to trigger on-demand revalidation?
4. **Developer Experience:** Keep API simple like Next.js
5. **Adapter Coordination:** Each adapter needs to implement ISR differently

---

## References

- [Next.js ISR Documentation](https://nextjs.org/docs/basic-features/data-fetching#incremental-static-regeneration)
- [Next.js On-Demand Revalidation Discussion](https://github.com/vercel/next.js/discussions/16488)
- [Vercel SvelteKit ISR Docs](https://vercel.com/docs/frameworks/sveltekit)
- [MDN Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control)

