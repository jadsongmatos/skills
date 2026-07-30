---
name: ralphy
description: "Configure and run Ralphy (github.com/michaelshimeles/ralphy) — an autonomous AI coding loop that drives Claude Code, OpenCode, Cursor, Codex, Qwen-Code, Factory Droid, GitHub Copilot, or Gemini CLI against a task list until every task is done. Use this whenever the user wants to turn a feature idea, PRD, or backlog into a ralphy-compatible task file (Markdown checklist, YAML with parallel groups, JSON, or GitHub Issues), choose flags for single-task vs PRD-loop mode, parallel/worktree/sandbox execution, branch-per-task and PR workflows, project rules/boundaries in .ralphy/config.yaml, run it non-blocking via tmux (including several ralphy invocations at once), or debug a stuck/failing run. Trigger on: ralphy, ralphy.sh, ralphy --prd, ralphy --parallel, .ralphy/config.yaml, tmux, autonomous coding loop, agentic PRD runner, run multiple coding agents at once, run tasks until done."
---

# Ralphy

[michaelshimeles/ralphy](https://github.com/michaelshimeles/ralphy) is an autonomous AI coding loop: point it at one task, or a whole list of them, and it repeatedly drives an AI coding CLI against the repo — writing code, running tests/lint, committing — one task at a time, until the list is done.

Use this skill to turn what the user wants built into a task file ralphy can consume, pick the right engine and flags, run it, and debug it if a run gets stuck or fails.

This assumes `ralphy`, the chosen AI engine's CLI (Claude Code by default — see the engine table in `references/flags.md`), and `tmux` are already installed and on PATH; setting any of those up is outside this skill's scope (Troubleshooting has the one-liner for the rare case `ralphy` itself turns out to be missing). Substitute `./ralphy.sh` for `ralphy` throughout if the user's on the cloned bash version instead of the npm one — both accept identical flags.

**Before running `ralphy` for real (anything without `--dry-run`), confirm the exact command with the user first.** It hands an AI agent unattended, permission-prompt-free access to the repo (see "Engines" below) — it will edit files, run whatever shell commands the agent decides on, and depending on flags, commit and open PRs, for as long as the task list takes. Running several invocations at once (Step 5) means several such agents unattended in parallel. Drafting the task file is safe to do proactively; actually kicking off the loop — one session or several — is a real action on their repo and deserves a nod first.

---

## Step 1: Single task, or a list?

- **One well-scoped change** — "add dark mode", "fix the auth redirect bug": brownfield mode, no file needed:
  ```bash
  ralphy "add dark mode toggle to the settings page"
  ```
- **Several related changes**: write a task file (Step 2) and run the loop:
  ```bash
  ralphy --prd PRD.md        # or --yaml / --json / --github — see below
  ```
  With no source flag, `ralphy` looks for `PRD.md` in the current directory by default.

If the user already described multiple things to build earlier in this conversation, don't make them repeat it — draft the task file from what they said and show it before running anything.

## Step 2: Write the task file

**Break the work into micro-tasks.** Each task becomes one isolated prompt to the AI engine, with no memory of any other task in the run — so a task has to be small enough to finish in one shot. If it can't be described in two or three sentences, split it. "Implement authentication" → separate tasks for the model, password hashing, signup endpoint, login endpoint, session handling, logout.

Pick a format:

**Markdown (default)** — simplest, good for a straight list:
```markdown
## Tasks
- [ ] create User model with email and password fields
- [ ] add password hashing utility function
- [ ] create signup API endpoint
```
Any unchecked `- [ ]` under any heading counts as a task; ralphy flips it to `- [x]` as each one finishes, top to bottom. For a large project, point `--prd` at a **folder** instead of a file — ralphy aggregates tasks across every `.md` file in it and updates completion in whichever file each task came from.

**YAML (`--yaml file.yaml`)** — supports explicit parallel groups (JSON does too, below; Markdown and GitHub Issues don't):
```yaml
tasks:
  - title: Create User model
    completed: false
    parallel_group: 1     # omit or 0 = sequential; same number = can run together
  - title: Create Post model
    completed: false
    parallel_group: 1
  - title: Add relationships between models
    completed: false
    parallel_group: 2     # runs only after group 1 finishes
```

**JSON (`--json file.json`)** — ralphy's real schema is a flat array:
```json
{
  "tasks": [
    { "title": "create auth", "completed": false, "parallel_group": 1, "description": "optional extra detail" }
  ]
}
```
`title` doubles as the task's unique ID (must be unique in the file), `parallel_group` is snake_case, and that's the whole shape. **This is not the same schema some other "Ralph"-technique tools use** — if a task file shows up with a `userStories` array, `acceptanceCriteria`, `passes`, or a `priority` int, that's a different tool's format; flatten it into the `tasks` array above before handing it to `--json`.

**GitHub Issues (`--github owner/repo [--github-label ready]`)** — no file; ralphy pulls open issues (optionally filtered by label) as the task list directly.

## Step 3: Pick the engine and the flags that matter most

```bash
ralphy --prd PRD.md                                    # Claude Code (default)
ralphy --prd PRD.md --opencode                          # OpenCode
ralphy --prd PRD.md --cursor   # or --codex / --qwen / --droid / --copilot / --gemini
```
`--model <name>` overrides the default model for whichever engine is active (`--sonnet` is shorthand for `--claude --model sonnet`). Anything after a bare `--` passes straight through to the engine's own CLI, e.g. `ralphy --copilot -- --allow-all-tools`.

The flags most worth knowing before a first run (full table in `references/flags.md`):

| Want | Flag |
|---|---|
| See the plan without touching anything | `--dry-run` |
| A bounded first run, not the whole backlog | `--max-iterations N` |
| Skip tests/lint for speed | `--no-tests`, `--no-lint`, or both via `--fast` |
| Run independent tasks concurrently | `--parallel` (`--max-parallel N`, default 3) |
| A branch + PR per task instead of committing straight to the branch | `--branch-per-task --create-pr` (`--draft-pr` for drafts) |

`--parallel` gives each agent its own git worktree + branch, then auto-merges back (AI resolves conflicts) unless `--create-pr` (keep as PRs) or `--no-merge` (keep the branches, merge/PR nothing) is set. **How tasks get batched together depends on the source**: with YAML/JSON, only tasks sharing an explicit `parallel_group` run concurrently — anything ungrouped (group 0, the default) always runs alone, one at a time. Markdown and GitHub Issues have no group concept, so `--parallel` there just grabs the next `--max-parallel` incomplete tasks in list order and runs them together — only safe if neighboring tasks in the file are actually independent of each other. If they're not, either reorder the file or switch to YAML/JSON and group explicitly. On a big repo where copying `node_modules` per worktree is too slow, add `--sandbox` — it symlinks read-only dependency directories and only copies the source files agents might touch; parallel mode also falls back to this automatically if worktree creation fails.

## Step 4: Project config (optional, worth it for repeat use)

```bash
ralphy --init                              # auto-detects language/framework/test/lint/build
ralphy --add-rule "use server actions, not API routes"
ralphy --config                            # view the current config
```
Writes `.ralphy/config.yaml` (rules + boundaries apply to every task, single or loop) and `.ralphy/progress.txt` (an auto-updated log — never point an agent at editing this file or the task file itself; ralphy already tells it not to). See `references/flags.md` for the full schema.

Auto-detection only recognizes Node/TypeScript, Python, Go, and Rust (via `package.json` / `pyproject.toml` / `go.mod` / `Cargo.toml`). For anything else — embedded C, firmware, other stacks — `--init` still creates the file, just with blank `test`/`lint`/`build` commands; fill those in by hand afterward.

## Step 5: Run it

```bash
ralphy --dry-run --prd PRD.md      # sanity-check the plan first
```
Confirm with the user before running for real (see the warning at the top), then launch it in **tmux** rather than a blocking foreground call — a PRD-loop run can take a long time, and tying up the conversation on one synchronous `bash` call for its whole duration is worse than checking in periodically. Look for a connected MCP tmux server first (session/pane creation, send-keys, capture-pane tools) and prefer it; otherwise drive `tmux` directly:

```bash
# one invocation = one named, detached session
tmux new-session -d -s ralphy-<slug> "ralphy --prd PRD.md --opencode; echo EXIT_$?"

# check on it later without blocking
tmux capture-pane -t ralphy-<slug> -p | tail -n 40

# everything currently running
tmux list-sessions 2>/dev/null | grep ^ralphy-
```
This is also how to run **several ralphy invocations at once** — different repos, unrelated single tasks, separate task lists — one named session each. That's concurrency *across* invocations, a different axis from ralphy's own `--parallel` (which fans out tasks *within* one invocation, via worktrees) — the two stack fine: `tmux new-session -d -s ralphy-backend "ralphy --prd PRD.md --parallel"` is one non-blocking session whose own tasks also run concurrently. Full tmux command set (listing, killing, checking exit status) in `references/flags.md`.

Each task becomes one prompt: project context, code-change rules, a boundaries list, a pointer to any `.claude/skills/`, `.opencode/skills/`, `.github/skills/`, or `.skills/` directory in the repo (ralphy asks the engine to read relevant skills from there before coding — so skills placed in the target repo, including this one, get picked up automatically during a run), then the task itself, then step-by-step instructions ending in test → lint → commit as configured. A failed task retries up to `--max-retries` (default 3) before ralphy moves on; rate-limit, quota, and auth/config errors are detected and abort early rather than burning through every retry.

### Quick recipes
```bash
# New repo, build a feature list, one PR per task
ralphy --init
ralphy --prd PRD.md --branch-per-task --create-pr

# Fast, low-ceremony single fix
ralphy "fix the off-by-one in pagination" --fast --opencode

# Big backlog — bounded first run before trusting it with the rest
ralphy --prd PRD.md --dry-run
ralphy --prd PRD.md --max-iterations 3

# Several independent runs at once, each in its own tmux session
tmux new-session -d -s ralphy-frontend "cd ~/proj/frontend && ralphy --prd PRD.md"
tmux new-session -d -s ralphy-backend  "cd ~/proj/backend  && ralphy --prd PRD.md"
```

## Troubleshooting

- **`ralphy: command not found`** — shouldn't come up (this skill assumes it's installed), but if it does: `npm install -g ralphy-cli`, or clone the repo and use `./ralphy.sh` (needs `jq`).
- **`<engine> CLI not found`** — the AI engine's own CLI isn't on PATH; not a ralphy problem, and it surfaces immediately since ralphy checks PATH before starting. Install/authenticate that engine's CLI first.
- **Stuck retrying the same task** — check `.ralphy/progress.txt` for what the engine actually attempted, or `tmux capture-pane` the session if it's running detached.
- **Parallel run failed to create worktrees** (e.g. nested-repo setups) — ralphy already falls back to sandbox mode automatically; if it's still failing, pass `--sandbox` explicitly.
- **tmux session name already taken** — `tmux kill-session -t <name>` first, or pick a more specific slug; comes up fast once several runs are launched the same day.
- **Flags don't match reality** — `ralphy --help` is authoritative over this file if they've drifted; ralphy ships frequent point releases (this skill reflects v4.7.2).

For the complete flag table, exact per-engine invocation (including which permission flag each one passes), and the full `.ralphy/config.yaml` schema, read `references/flags.md`.
