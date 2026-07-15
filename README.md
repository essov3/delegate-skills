# delegate-skills

[![skills.sh](https://skills.sh/b/amElnagdy/delegate-skills)](https://skills.sh/amElnagdy/delegate-skills)

Skills for **delegating coding work to a separate CLI agent and landing it yourself**. Your agent (the
orchestrator) writes a self-contained brief, hands it to an implementer CLI, then reviews the diff and
commits — staying the reviewer the whole way.

Three skills ship today: **`codex-delegate`** drives the OpenAI Codex CLI, **`opencode-delegate`** drives
the OpenCode CLI, and **`grok-delegate`** drives Grok Build headlessly. Same loop, different implementer.

## Install

Browse first:

```bash
npx skills add amElnagdy/delegate-skills --list
```

Install the package, or just one skill:

```bash
npx skills add amElnagdy/delegate-skills
npx skills add amElnagdy/delegate-skills --skill codex-delegate
npx skills add amElnagdy/delegate-skills --skill opencode-delegate
npx skills add amElnagdy/delegate-skills --skill grok-delegate
```

Install for a specific agent, or globally:

```bash
npx skills add amElnagdy/delegate-skills --skill codex-delegate --agent claude-code
npx skills add amElnagdy/delegate-skills --global
```

Works with any orchestrating agent the [Skills CLI](https://github.com/vercel-labs/skills) supports.

## What it does

The loop:

1. **Write a brief** — a self-contained task spec; the implementer sees only what you send.
2. **Dispatch** it with the bundled `relay.mjs`.
3. **Wait** for completion — the helper writes a structured `result.json`.
4. **Review** the diff — re-run the project's gates yourself; pair with [guard skills](https://github.com/amElnagdy/guard-skills).
5. **Land** it — *you* commit, because committing belongs to the reviewer.

```text
Use $codex-delegate to have Codex implement the refactor in services/billing/, then review and commit it.
Use $grok-delegate to have Grok Build implement the migration, then review the diff and land it yourself.
```

## How this differs from the OpenAI Codex plugin

The official openai-codex Claude Code plugin is excellent and **complementary** — this skill builds on
the same `codex` CLI, it doesn't replace the plugin. They point in different directions:

- The plugin's `codex:codex-rescue` agent is a **forwarder**: it hands one task to Codex and returns the output.
- The plugin's review command and stop-review gate run the **inverse** direction: **Codex reviews your work**.
- `codex-delegate` is the orchestration loop in the other direction: brief → dispatch → poll → review → commit.

If you have the plugin installed, its companion CLI is an optional alternative dispatch backend; the
bundled `relay.mjs` is the default because it needs nothing but the `codex` binary.

## The skills

### codex-delegate

Drive the OpenAI Codex CLI as a background implementer: write the brief, dispatch via `relay.mjs`,
review the diff, commit it yourself.

### opencode-delegate

Drive the OpenCode CLI as a background implementer. Autonomy is selected through the OpenCode agent:
`build` for implementation and `plan` for read-only review.

### grok-delegate

Drive Grok Build through its documented headless mode with newline-delimited `streaming-json` output,
session resume/continue support, and a stable relay `result.json`. Implementation runs auto-approve tool
calls by default; read-only runs remove write/edit tools, avoid auto-approval, and detect working-tree
status changes. Raw events are retained so CLI output changes remain diagnosable.

### gemini-delegate

*Planned.* A delegate skill for the Gemini CLI, if and when it gains a comparable non-interactive mode.

## Requirements

- For `codex-delegate`: the [`codex` CLI](https://github.com/openai/codex), authenticated with `codex login`.
- For `opencode-delegate`: the [`opencode` CLI](https://opencode.ai), authenticated with `opencode auth login`.
- For `grok-delegate`: the [Grok Build CLI](https://docs.x.ai/build/cli/headless-scripting), authenticated with `grok login`.
- Node 18+ and `git`.
- An orchestrating agent that can run shell commands and read files.
- Shell examples assume bash/zsh (macOS/Linux, or Git Bash/WSL on Windows).

## Trust and validation

This package is intentionally inspectable:

- All skill content is Markdown, plus exactly **one** executable per skill — each a `scripts/relay.mjs`.
- Each relay uses Node built-ins only, makes no network calls itself, reads or writes no credentials, and
  shells out only to its implementer CLI and `git`.
- Relays never commit — committing is always the orchestrator's job after review.

**Verification status:** Codex and OpenCode retain their existing verification status. The new Grok relay
has implementation-level validation for argument handling, result generation, missing-binary behavior,
signal reporting, raw event preservation, and read-only git-status detection. A real authenticated Grok
run remains required before claiming end-to-end verification against a specific Grok CLI version.

## Repository shape

```text
skills/
├── codex-delegate/
├── opencode-delegate/
├── grok-delegate/
│   ├── SKILL.md
│   ├── scripts/relay.mjs
│   └── references/
│       ├── writing-the-brief.md
│       ├── dispatch-and-poll.md
│       ├── review-and-land.md
│       └── multi-task-queues.md
└── gemini-delegate/ (planned)
```

The `SKILL.md` stays small so it loads cheaply; references load only when needed.

## License

MIT — see [LICENSE](LICENSE).
