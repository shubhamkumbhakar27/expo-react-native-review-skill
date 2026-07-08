---
name: expo-react-native-pr-review
description: Review Expo / React Native pull requests (expo-router, React Query, NativeWind) against a weighted, reliability-first rubric — request timeouts, data-first rendering, feature-first architecture & layer direction, React Query discipline, server state, TypeScript strictness, error handling, images, styling, and offline/security. Produces a score and a merge recommendation. Use when reviewing a pull request, diff, or code changes in a React Native / Expo app, or when the user asks for a code review, PR review, or audit of mobile app code.
---

# Expo / React Native PR Review

A 10-rule weighted rubric for reviewing **Expo / React Native** pull requests, packaged as an Agent Skill. It scores a diff against ten rules covering the three failure modes RN apps are most prone to — **latency**, **availability** (white/blank screens, hangs), and **broken offline behavior** — plus feature-first architecture, type safety, and styling. Each rule has a **penalty** per violation and a **reward** for correct implementation; the review ends with a **net score** and a **merge recommendation**.

**Stack assumptions:** `expo` (expo-router), `react-native`, `@tanstack/react-query`, `nativewind`, a persisted offline-first query cache, and native SDKs (e.g. Sendbird). Rules degrade gracefully for any RN app; adjust the examples to your stack.

## Scoring rubric

| #  | Rule                                                  | Penalty / occurrence | Reward |
| -- | ----------------------------------------------------- | -------------------- | ------ |
| 1  | Bounded network requests (timeout/abort)              | −1000                | +10    |
| 2  | Data-first rendering (no blank screens)               | −500                 | +50    |
| 3  | Feature-first architecture, layer direction, thin routes | −200              | +50    |
| 4  | React Query key + cache discipline                    | −100                 | +50    |
| 5  | Server state in React Query only                      | −100                 | +50    |
| 6  | Error boundaries, no `console.log`/empty catch        | −200                 | +50    |
| 7  | TypeScript strictness (no `any`, typed SDKs)          | −100                 | +50    |
| 8  | Remote images via `expo-image`                        | −50                  | +50    |
| 9  | Styling: design tokens / NativeWind, no magic         | −50                  | +50    |
| 10 | Offline & secrets safety                              | −200                 | +50    |

**Critical blockers:** Rules **#1** (unbounded request) and **#2** (a screen that can render blank) **block the merge regardless of net score** — these are the direct causes of the app-hang and white-screen reports. Flag them as 🔴 and set the recommendation to Block.

## Review workflow

Copy this checklist and work through it:

```
- [ ] Step 1: Get the diff and list changed files (only review changed lines + their direct context)
- [ ] Step 2: Classify each changed file by layer (route / screen / feature / hook / lib / ui)
- [ ] Step 3: Score each applicable rule, reading references/NN-*.md for detection patterns
- [ ] Step 4: Record every occurrence with file:line, penalty/reward, and a concrete fix
- [ ] Step 5: Apply critical blockers (#1, #2), compute the net score, write the report
```

**Step 1 — Get the diff.** Prefer `git diff <base>...HEAD` (or the PR diff via `gh pr diff <n>`). Review the changed hunks; only flag pre-existing issues if the PR touches those lines.

**Step 2 — Classify.** `app/**` = routes (must be thin — import one page/feature component and render it, never define the screen inline), `src/components/pages/**` or `src/features/<feature>/**` = the page/screen components a route delegates to (own their state/hooks/JSX), `src/shared/**` (`design`, `atoms`, `molecules`, `organisms`, `components`) = cross-feature UI, `src/lib/**` (`endpoints`, `api`, `data-fetch`, `hooks`, `sendbird`, `storage`) = infrastructure with no UI, `src/providers/**` = composition. Rules apply differently per layer (see each reference).

**Step 3 — Score.** For each rule, open `references/NN-<rule>.md` for the detection patterns and bad/good examples. Only load the references you need.

**Step 4 — Record occurrences.** Each hit is one occurrence of that rule's penalty. Give the exact `file:line` and a one-line fix.

**Step 5 — Report.** Use the output format below.

## Rule index

1. [Bounded network requests](references/01-network-timeouts.md) — every `fetch`/request has an `AbortController` timeout.
2. [Data-first rendering](references/02-data-first-rendering.md) — loading/empty/error/ready states; never branch on `error` before cached `data`.
3. [Feature-first architecture, layer direction & thin routes](references/03-feature-architecture.md) — `app/routes → src/features → src/shared → src/lib`; **routes are thin** — a file under `app/` only maps a URL to a page/feature component (`return <XyzPage />`) and never defines the screen inline; features own their code and never import each other; files <300 lines.
4. [React Query discipline](references/04-react-query-discipline.md) — typed key factory, cache tiers, invalidation, refetch policy.
5. [Server state in React Query only](references/05-state-management.md) — no mirroring server data into `useState`/Context.
6. [Error handling & logging](references/06-error-handling-logging.md) — route `ErrorBoundary`, no `console.log`, no empty `catch`.
7. [TypeScript strictness](references/07-typescript-strictness.md) — `strict`, no `any`, typed SDK objects (no `as any`).
8. [Remote images via expo-image](references/08-images.md) — `expo-image` with `contentFit` + `recyclingKey`.
9. [Styling & design tokens](references/09-styling-design-system.md) — NativeWind + tokens; no magic hex/spacing; accent used sparingly.
10. [Offline & secrets safety](references/10-offline-and-security.md) — persisted cache, purge on logout, secrets in secure store.

## Output format

```markdown
# Expo/RN PR Review — <PR title or branch>

## Score: <net score>  →  <✅ Approve | 🟡 Approve with changes | 🔴 Block>

## Critical blockers
- 🔴 <Rule #> <file:line> — <one-line reason> (blocks merge)
  _(omit this section if none)_

## Findings
### 🔴 / 🟡 / 🟢 Rule <#>: <name> (<±points>)
- `<file>:<line>` — <what's wrong> → <concrete fix>

## Strengths
- <rule followed well, with file:line and reward>

## Summary
<2–3 sentences: overall risk, what to fix before merge>
```

**Severity mapping:** 🔴 Critical = rules #1/#2 or any net-negative rule cluster that risks a hang/blank screen; 🟡 Suggestion = other violations; 🟢 = rewards/strengths.

**Merge recommendation:**
- 🔴 **Block** — any critical blocker (#1 or #2) present, or net score < 0.
- 🟡 **Approve with changes** — net score ≥ 0 but with penalized findings.
- ✅ **Approve** — no penalties and at least the core rewards present.

## Customizing

The rubric is intentionally editable. Fork the repo, then adjust weights/thresholds in this `SKILL.md` and refine detection patterns or examples in `references/`. Common team-specific additions: a single import boundary for platform primitives (e.g. UI from `@/design`, runtime APIs from `@/native`, enforced by ESLint), a mandatory shared `<List>`/`<ScreenState>`, or a telemetry sink for `logApiCall`.
