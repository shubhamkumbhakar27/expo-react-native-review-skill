# Expo / React Native PR Review (Skill)

A 12-rule, reliability-first rubric for reviewing **Expo / React Native** pull requests, packaged as an Agent Skill. Works with Cursor, Claude Code, GitHub Copilot, Codex CLI, Gemini CLI, and any other Agent Skills–compatible host.

The skill scores a diff against twelve weighted rules targeting the failure modes RN apps hit most — hangs, blank/white screens, and broken offline behavior — plus architecture, type safety, and accessibility. Each rule has a penalty for violations and a reward for correct implementation; the agent produces a final **score** and **merge recommendation**.

## What it covers

| #  | Rule                                          | Penalty / occurrence | Reward |
| -- | --------------------------------------------- | -------------------- | ------ |
| 1  | Bounded network requests (timeout/abort)      | −1000                | +10    |
| 2  | Data-first rendering (no blank screens)       | −500                 | +50    |
| 3  | Layer direction & thin routes (<500 lines)    | −200                 | +50    |
| 4  | No work in `renderItem` / list performance    | −200                 | +50    |
| 5  | React Query key + cache discipline            | −100                 | +50    |
| 6  | Server state in React Query only              | −100                 | +50    |
| 7  | Error boundaries, no `console.log`/empty catch| −200                 | +50    |
| 8  | TypeScript strictness (no `any`, typed SDKs)  | −100                 | +50    |
| 9  | Remote images via `expo-image`                | −50                  | +50    |
| 10 | Styling: design tokens / NativeWind, no magic | −50                  | +50    |
| 11 | Offline & secrets safety                      | −200                 | +50    |
| 12 | Accessibility                                 | −50                  | +50    |

Rules **#1** and **#2** are critical and **block the merge** regardless of net score. See [`SKILL.md`](SKILL.md) for the full workflow and [`references/`](references/) for per-rule detection patterns, bad/good examples, and edge cases.

## How it works

Progressive disclosure: `SKILL.md` is a compact dispatcher with the scoring rubric, rule index, and review workflow. Each rule lives in its own self-contained file under `references/`, loaded only when scoring that rule.

```
expo-react-native-pr-review/     ← skill package (what agents load)
├── README.md
├── LICENSE
├── SKILL.md
└── references/
    ├── 01-network-timeouts.md
    ├── 02-data-first-rendering.md
    ├── 03-layer-architecture.md
    ├── 04-list-performance.md
    ├── 05-react-query-discipline.md
    ├── 06-state-management.md
    ├── 07-error-handling-logging.md
    ├── 08-typescript-strictness.md
    ├── 09-images.md
    ├── 10-styling-design-system.md
    ├── 11-offline-and-security.md
    └── 12-accessibility.md
```

## Publishing on GitHub

For `gh skill install <owner>/<repo>` to discover it, the skill must live at `skills/expo-react-native-pr-review/` from the **repo root**. Two options:

- **Dedicated repo (recommended):** create a new repo whose root contains a `skills/` folder, and move this package to `skills/expo-react-native-pr-review/`. Layout: `<repo>/skills/expo-react-native-pr-review/SKILL.md`.
- **This repo:** the package already sits at `skills/expo-react-native-pr-review/`, so pushing this repo works as-is.

## Install (Cursor)

**With the GitHub CLI** (v2.90.0+):

```bash
gh skill install <owner>/<repo> --agent cursor
```

This drops the skill into `.cursor/skills/expo-react-native-pr-review/` in the current project (Cursor is project-scoped only). Then run **`Developer: Reload Window`** from the command palette so Cursor picks it up.

**Manual install (no `gh`):**

```bash
git clone https://github.com/<owner>/<repo>.git /tmp/rn-review-skill \
  && mkdir -p .cursor/skills \
  && cp -R /tmp/rn-review-skill/skills/expo-react-native-pr-review .cursor/skills/ \
  && rm -rf /tmp/rn-review-skill
```

Other hosts: `gh skill install <owner>/<repo> --agent <name>` (Claude Code, Copilot, Codex CLI, Gemini CLI, …). Claude Code personal installs land in `~/.claude/skills/expo-react-native-pr-review/`.

## Verifying

Open your agent and ask something that should trigger the skill:

> "Review this PR against our React Native standards."

You should see the agent reference the rubric and produce a structured score with findings. If nothing happens: confirm the skill is at the expected path, reload the Cursor window, and make sure your prompt mentions PR review / code review / audit.

## Customizing

The skill is intentionally readable and editable:

1. Fork the repo.
2. Edit `SKILL.md` to adjust scoring weights, merge thresholds, or output format.
3. Edit files under `references/` to refine examples and detection patterns for your stack (Sendbird vs. other SDKs, FlatList vs. FlashList, etc.).
4. Republish under your own GitHub handle and install from there.

## Compatibility notes

- Pure markdown — no shell injection, no forked-subagent execution. Portable across every Agent Skills–compatible host.
- No `allowed-tools` frontmatter, so the skill doesn't pre-approve any tools; your agent's normal permission settings stay in control.

## License

MIT. See [LICENSE](LICENSE).
