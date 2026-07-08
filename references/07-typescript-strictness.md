# Rule 7 — TypeScript strictness

**Penalty −100/occurrence · Reward +50**

`strict` mode stays on. No `any` (explicit or implicit), no `as any`, no `@ts-ignore`/`@ts-expect-error` without a justification comment. External SDK objects (Sendbird messages, push payloads, deep-link params) are **typed at the boundary** — narrowed into app types, not cast blindly. Public functions/hooks have explicit return types.

## Why it matters
`any` and blind casts silently disable the compiler exactly where data is least trusted (network/SDK boundaries), which is where runtime crashes originate.

## Detection
- `: any`, `as any`, `Array<any>`, `Record<string, any>`.
- `@ts-ignore` / `@ts-expect-error` with no explanation.
- `(msg as UserMessage).message` — blind cast of an SDK union.
- `JSON.parse(...)` used without a validating narrow.
- Implicit-any params in callbacks (weakened by loose config).

## Bad
```ts
function onMessage(msg: any) {           // any at the SDK boundary
  const text = (msg as UserMessage).message;
}
```

## Good
```ts
function onMessage(msg: BaseMessage) {
  if (isUserMessage(msg)) {              // narrow, don't cast
    const text: string = msg.message;
  }
}
```

## Scoring
- −100 per `any`/blind cast/unjustified suppression at a data or SDK boundary.
- +50 for a typed boundary (type guard / parser) introduced in the diff.

## Edge cases
- `unknown` + a narrowing guard is the correct escape hatch, not `any`.
- Generated API types should be the source of truth; don't redeclare drifting shapes.
