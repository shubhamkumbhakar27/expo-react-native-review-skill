# Rule 11 — Config & environment validation (fail fast)

**Penalty −100/occurrence · Reward +50**

All external configuration (`EXPO_PUBLIC_*` env vars, `Constants.expoConfig.extra`) is read in **one module** and validated against a **schema (zod)** at startup. Required values (API base URL, SDK app ids) **fail fast** with a clear, aggregated error naming each missing/invalid var; optional values carry explicit safe defaults. Config is never read ad-hoc with `process.env.X` scattered across the codebase, and a required value is never silently coerced to `""` — an empty base URL becomes mysterious 404s app-wide, a missing SDK id becomes a crash deep inside a screen.

## Why it matters
A misconfigured build should surface as a single, descriptive boot error — not as latent 404s, a blank chat tab, or a `TypeError` three screens in. Centralizing + validating config also documents the contract (which vars exist, which are required) and makes the app impossible to start in a half-configured state.

## Detection
- `process.env.EXPO_PUBLIC_*` or `Constants.expoConfig?.extra?.x` read directly in feature/lib files instead of a single `env` module.
- Required config typed as `string` but defaulted with `?? ""` / `|| ""`, so a missing value passes silently.
- No schema/validation of env at startup (values trusted blindly).
- A base URL / auth key / SDK id that can be empty at runtime with no guard.

## Bad
```ts
// scattered, unvalidated, silently-empty
const apiUrl = process.env.EXPO_PUBLIC_API_URL ?? "";   // "" → every request 404s
fetch(`${apiUrl}/event`);
```

## Good
```ts
// src/shared/constants/env.ts — one module, validated once at import
import { z } from "zod";
const schema = z.object({
  apiUrl: z.string().refine((v) => /^https?:\/\/.+/.test(v), "must be an http(s) URL"),
  sendbirdAppId: z.string().min(1, "is required for chat"),
  env: z.string().min(1),
});
const parsed = schema.safeParse(rawEnv);
if (!parsed.success) {
  const details = parsed.error.issues.map((i) => `  • ${String(i.path[0])}: ${i.message}`).join("\n");
  throw new Error(`Invalid environment configuration — the app cannot start:\n${details}`);
}
export const env = parsed.data;   // typed, guaranteed-present
```

## Scoring
- −100 per required config value read unvalidated / defaulted to empty, or per env var read directly outside the central `env` module.
- +50 when the diff adds/uses a single validated `env` module that fails fast on missing required vars.

## Edge cases
- `EXPO_PUBLIC_*` values are inlined at build time; validate the resolved values (from `Constants.expoConfig.extra` / `process.env`), not raw `.env` parsing.
- Only *required* vars throw; genuinely optional vars (store URLs, min version) get documented defaults, never a hard failure.
- Validate once at module import so the failure happens at startup, not lazily on first use.
