# Rule 1 — Bounded network requests (timeout / abort)

**Penalty −1000/occurrence · Reward +10 · CRITICAL BLOCKER**

Every network request must be time-bounded with an `AbortController`. An unbounded `fetch` on a hung backend freezes the screen indefinitely (splash, chat, detail) — the single biggest availability bug in RN apps. There must be exactly **one** HTTP client that owns the timeout; per-call `fetch` duplication is banned.

## Why it matters
A request that never resolves and never rejects leaves the UI stuck on a spinner forever. There is no recovery without a timeout that converts the hang into a typed, catchable error.

## Detection
- `fetch(` / `axios(...)` calls with **no** `signal` / `AbortController` / timeout.
- New `fetch` logic inside a component, hook, or endpoint file instead of going through the shared client.
- A client that has `signal` support but a call path that bypasses it.
- Timeouts set but never `clearTimeout`-ed (leaks) — minor, still flag.

## Bad
```ts
// no timeout — hangs forever if the server stalls
export async function getEvents() {
  const res = await fetch(`${BASE}/events`);
  return res.json();
}
```

## Good
```ts
// lib/http/client.ts — one client, every request bounded
export async function api<T>(path: string, init: RequestInit = {}, timeoutMs = 15000): Promise<T> {
  const controller = new AbortController();
  const timer = setTimeout(() => controller.abort(), timeoutMs);
  try {
    const res = await fetch(`${BASE}${path}`, { ...init, signal: controller.signal });
    if (!res.ok) throw new ApiError(res.status);
    return (await res.json()) as T;
  } finally {
    clearTimeout(timer);
  }
}
```

## Scoring
- −1000 for each request path that can hang unbounded (a critical blocker on its own).
- +10 when the diff routes network calls through the single timed client.

## Edge cases
- Streaming/long-poll endpoints may use a longer, explicit timeout — acceptable if intentional and documented.
- The transport must **not** navigate or show alerts on error; surface a typed error and let providers react (see rule 6).
- Connectivity/timeout errors must **never** clear the session — only a real 401 on the auth probe may (see rule 10).
