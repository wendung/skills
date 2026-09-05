---
name: wendung-sdk
description: Install, configure, and use @wendung/sdk, the browser SDK for Wendung funnel and web analytics. Use when adding Wendung to a web project (Next.js, React, Vue, or any browser app), instrumenting funnel steps, turning on pageview tracking, identifying users, debugging event delivery, or answering questions about Wendung funnels, conversion, drop-off, and site traffic. Covers init options, batching and retry behavior, identity and session rules, framework patterns, and the Wendung MCP servers for live analytics and docs search.
---

# Wendung SDK

Wendung is funnel and web analytics. Apps send named step events, and the dashboard turns them into funnels with conversion, drop-off, and timing data. Full pageviews feed visitor, session, page, source, location, and device reports per site. Aggregate pageviews contribute pageview counts without visitor or session counts. `@wendung/sdk` is the browser SDK.

The SDK is browser-only. Never call it from server code (Next.js Server Components, Route Handlers, Server Actions, SSR). Importing it on the server is safe (no import-time side effects); calling it is not.

## Docs and MCP servers

Full docs: https://docs.wendung.app. Append `.md` to any page URL for raw markdown.

Wendung provides two MCP servers. Everything in this skill works without them, but use them when they are connected:

| Server | URL | Use for | Sign-in |
|---|---|---|---|
| `wendung` | https://api.wendung.app/mcp | Workspaces, projects, funnels, and live analytics: `list_workspaces`, `list_projects`, `list_funnels`, analytics tools, `get_pageview_summary`, `get_pageview_timeseries`, `get_pageview_breakdown`, `get_ingest_health` | OAuth |
| `wendung-docs` | https://docs.wendung.app/mcp | Searching the Wendung docs | None |

- If the `wendung` server is connected, use it for anything involving the user's real data: verifying that events arrive, listing funnels, and answering conversion and drop-off questions with real numbers. Write tools (creating or editing funnels and dashboards) only work if the user granted write access at sign-in.
- If the `wendung-docs` server is connected, search the docs through it. Otherwise fetch pages from https://docs.wendung.app with `.md` appended.
- If neither server is connected and the task would benefit from one, offer to register them. With the user's permission, fetch https://docs.wendung.app/prompt.md and follow the registration steps for your agent. The `wendung` server requires OAuth sign-in, and most agents need a restart before newly registered MCP tools appear; tell the user both.

## Install and initialize

```bash
npm install @wendung/sdk
```

```ts
import { Wendung } from '@wendung/sdk'

Wendung.init({ apiKey: 'pk_...' })
```

Sites without a build step (plain HTML, Hugo, Jekyll, Webflow, WordPress) load the same SDK from the CDN instead. The tag initializes the SDK from its data attributes and exposes `Wendung` on `window`; `data-track-pageviews` accepts a bare attribute or `"true"` for consent-required pageviews, `"auto"` for aggregate pageviews while pending and full pageviews after consent, or `"false"` to disable automatic pageviews. `data-debug` is a presence flag; `data-endpoint` overrides the ingest URL. Use `defer`, never `async`, and give your own scripts that call `Wendung` `defer` too. `/sdk/v0/wendung.js` tracks the latest 0.x release; `/sdk/<version>/wendung.js` pins one. Guide: https://docs.wendung.app/guides/script-tag

```html
<script defer src="https://cdn.wendung.app/sdk/v0/wendung.js" data-api-key="pk_..." data-track-pageviews></script>
```

The publishable key comes from Settings → API Keys in the Wendung dashboard. In projects, read it from an env var (`NEXT_PUBLIC_WENDUNG_KEY`, `VITE_WENDUNG_KEY`, or equivalent). If no key is available, ask the user for it; do not invent one.

Call `init()` once at app startup, client-side, before any other Wendung call. Every other method throws `[Wendung] Call .init() first` until it runs. `init()` is re-entrant: calling it again tears down the previous instance first, so React Strict Mode and HMR are safe.

### init() options

| Option | Default | Notes |
|---|---|---|
| `apiKey` | required | Sent as the `X-API-Key` header. Falsy throws `Missing required configuration`. |
| `endpoint` | `https://ingest.wendung.app/v1/events` | Omit unless self-hosting. Must be https (http allowed for localhost). Passing `endpoint: undefined` explicitly overrides the default and throws. |
| `flushInterval` | `5000` (ms) | Timer between automatic flushes. Not validated. |
| `maxBatchSize` | `50` | Effective value clamped to 1–100; the ingest endpoint accepts at most 100 events per request. |
| `debug` | `false` | Adds `[Wendung]` lifecycle logs. `[Wendung - Warning]` messages always log. |
| `trackPageviews` | `false` | `false` or omitted disables automatic pageviews. `true` enables full pageviews only with granted consent. `'auto'` enables aggregate pageviews while pending, then full pageviews after consent. Denial stops both. Tracks the initial load and history navigation; aggregate collection ignores changes to only the query string or fragment. Pageviews use the plan's pageview allowance and appear in reports for registered sites. |
| `consent` | `'granted'` | `'pending'` allows only aggregate pageviews with `trackPageviews: 'auto'`; otherwise it stops collection. It does not read or write browser storage. `'denied'` stops all collection, discards queued events, and removes stored SDK identifiers. The app must pass its saved choice on every load. Script tag: `data-consent="pending"`. |

## API

| Method | Behavior |
|---|---|
| `Wendung.step(name, properties?)` | Queues an event. `name`: string, 1–100 chars. `properties`: plain object only. Invalid input logs a warning and the call is ignored; nothing throws. |
| `Wendung.page()` | Queues a `$pageview` for the current URL. Only needed with `trackPageviews` off, or for routers that change the URL without the history API. Calling it on a navigation the SDK already observed records the page twice. |
| `Wendung.identify(userId, traits?)` | Replaces identity wholesale, traits included. No merging, so pass the full trait set every time. Identity persists in localStorage and is restored by `init()`. Identifying a different user starts a new session; the first identify in a session keeps it, and the session's earlier anonymous events are attributed to that user at query time. |
| `Wendung.flush()` | Sends at most one batch per call. Resolves after the request settles; never rejects. |
| `Wendung.sendBeacon()` | Drains the entire queue as parallel `fetch(..., { keepalive: true })` requests. Despite the name it never uses `navigator.sendBeacon`. This is what runs automatically on page hide. |
| `Wendung.reset()` | Call on logout. Flushes pending events first (delivered with the identity they were tracked under), then starts a new session and clears identity including the persisted copy. |
| `Wendung.destroy()` | Stops the timer and listeners, then fires one final non-awaited batch; remaining events are lost. If delivery matters, `await Wendung.sendBeacon()` first. After destroy, every call throws until `init()` runs again. |
| `Wendung.setConsent(state)` | `'granted'` starts full analytics. After pending aggregate collection, it sends the queued aggregate batch without recounting the current page or linking earlier pageviews to the visitor. Call `identify()` again if the user is known. `'denied'` stops collection, discards the queue, and clears SDK identifiers. `'pending'` discards the current queue and leaves storage untouched, collecting only aggregate pageviews if `trackPageviews` is `'auto'`. Repeating the current state does nothing. |
| `Wendung.getConsent()` | Returns the current state. Do not guard `page()` or `flush()` on granted consent if pending aggregate collection is intended. The SDK enforces the configured behavior. |

## Pageviews and consent

Use this configuration when the user wants aggregate pageviews before a choice and full analytics after consent:

```ts
Wendung.init({
  apiKey: 'pk_...',
  trackPageviews: 'auto',
  consent: 'pending',
})
```

Restore the saved choice on later loads and call `setConsent()` from the consent manager. Keep `trackPageviews: true` when the user wants no collection before consent. Do not add a separate `pageviewMode` option.

Pending aggregate collection drops `identify()` and custom events. Pageviews contain origin and pathname, origin-only referrer, timestamps, and SDK metadata. They omit visitor and session IDs, query parameters, fragments, title, and screen size. Ingest retains country and parsed browser, OS, and device type. No IP hash is used to identify visitors. Pathnames can still contain personal information; this mode does not establish a consent exemption.

Pageview totals include aggregate and consented traffic. Unique visitors, current visitors, sessions, bounce rate, duration, views per session, and entry and exit pages cover consented traffic only. Aggregate views are not estimates of unique visitors. Changing collection settings does not remove earlier visitors from a report's date range.

## Runtime behavior

- Flush triggers: a 5 s timer (browser only), the queue reaching the batch size, `visibilitychange` to hidden, `pagehide`, and manual calls.
- Retry: a 4xx response drops the batch permanently. 5xx and network errors requeue it at the front with the same `batchId`. No backoff, no retry cap.
- The queue caps at 1000 events; overflow drops the oldest events.
- With granted consent, sessions use a UUID persisted in `sessionStorage` under `@wendung/sdk/session/v1`, with a 30 minute idle timeout and 24 hour maximum. Identity is separate, persisted in `localStorage` under `@wendung/sdk/identity/v1`, and restored on `init()`.
- There is no payload byte limit; the only cap is 100 events per request.

## Framework patterns

### Next.js (App Router)

Initialize in a client provider; the root layout stays a server component.

```tsx
// app/providers/wendung-provider.tsx
'use client'

import { useEffect } from 'react'
import { Wendung } from '@wendung/sdk'

export function WendungProvider({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    Wendung.init({ apiKey: process.env.NEXT_PUBLIC_WENDUNG_KEY! })
    return () => Wendung.destroy()
  }, [])

  return <>{children}</>
}
```

Wrap `children` with `<WendungProvider>` in `app/layout.tsx`. For web analytics, choose `trackPageviews: true` or `'auto'` as described above; the SDK follows App Router navigations on its own. Identify from a null component that watches the auth session. Full guide: https://docs.wendung.app/guides/nextjs.md

### React SPA (Vite, React Router)

Call `Wendung.init()` at the entry point before rendering, with `trackPageviews: true` or `'auto'` if the user wants web analytics; React Router navigations are picked up automatically. Full guide: https://docs.wendung.app/guides/spa.md

### Plain JS or other frameworks

Same rules everywhere: init once at startup in browser code, `step()` from event handlers, `identify()` after login, `reset()` on logout.

## Instrumenting funnels well

- Use stable snake_case action names that match funnel steps in the dashboard: `signup_completed`, `checkout_started`. This is a convention, not enforced by the SDK.
- When the app needs consent for analytics (EU visitors, an existing cookie banner), init with `consent: 'pending'`, restore the saved choice on later loads, and call `setConsent()` from the banner's callbacks. Do not delay `init()` instead: tracking calls throw before `init()`. Pending consent drops custom events and identification; it collects aggregate pageviews only with `trackPageviews: 'auto'`. Guide: https://docs.wendung.app/guides/consent.md
- Do not model pageviews as steps. Funnels are built from actions (`signup_completed`, `checkout_started`); traffic comes from `trackPageviews`, which the dashboard reports per site.
- Properties carry event-specific context. User-level data belongs in identify traits.

## Debugging

Set `debug: true` and watch the console. `[Wendung - Warning]` messages tell you what happened: a 4xx status means the key or payload is wrong and the batch was dropped; re-queuing messages mean transient network or 5xx failures; `fetch is unavailable` means an unsupported environment. Verify end-to-end delivery in the dashboard, or through the Wendung MCP server's `get_ingest_health` tool if it is connected. Full reference: https://docs.wendung.app/configuration/debugging.md

A `403` is the one status that has nothing to do with the key or the payload: the page's origin is not on the project's allowed origins list. A project with an empty list accepts every origin, so this only appears once someone adds the first one, and from then on `http://localhost:3000` is rejected along with everything else. Tell the user to turn on Dev mode in Settings → Allowed origins. It accepts loopback origins and requests sent without an `Origin` header for 24 hours, then switches itself off. The change takes up to a minute to reach the ingest endpoint.
