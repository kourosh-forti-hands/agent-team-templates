# Agent Team Execution Templates

Parameterized templates for orchestrating [Claude Code](https://docs.anthropic.com/en/docs/claude-code) agent teams to build software projects end-to-end — from research and technology selection through implementation, deployment, and validation.

## What's in this repo

| File | Purpose |
|------|---------|
| [`AGENT_TEAM_TEMPLATE.md`](AGENT_TEAM_TEMPLATE.md) | The core template. A parameterized execution guide with `{{PLACEHOLDER}}` fields that get populated for your specific project. Defines phases (Bootstrap, Assessment, Foundation, Features, Deployment, Validation) and three team topologies (solo, squad, swarm). |
| [`AGENT_TEAM_POPULATE_GUIDE.md`](AGENT_TEAM_POPULATE_GUIDE.md) | The interview prompt. Paste it into Claude Code and it runs a structured 7-round interview to populate the template — asking about your project identity, tech stack, architecture, features, team topology, success criteria, and phase details. Includes prerequisites and setup instructions. |
| [`RALPH_LOOP_TEMPLATE.md`](RALPH_LOOP_TEMPLATE.md) | The original loop-based template that inspired this project. Included as a reference for the alternative execution model (iterative loops with promise-based completion detection). |

## Quick start

1. Clone or download this repo into your project root
2. Open Claude Code in your project directory
3. Copy the **Kickoff Prompt** from `AGENT_TEAM_POPULATE_GUIDE.md` (everything between the `---START---` and `---END---` markers)
4. Paste it into Claude Code
5. Answer the 7 rounds of questions
6. Claude Code produces a populated `AGENT_TEAM_EXECUTION_GUIDE.md` tailored to your project

See the [Prerequisites section](AGENT_TEAM_POPULATE_GUIDE.md#prerequisites) in the populate guide for detailed setup instructions, including optional MCP servers and skills that enhance capabilities.

## Team topologies

The template supports three execution strategies:

- **Solo** — No teams. The lead spawns sequential `Task()` calls for each phase. Cheapest, simplest. Best for small projects.
- **Squad** — One persistent team for the entire project. All tasks created upfront with dependencies. Teammates spawned as tasks become unblocked.
- **Swarm** — Per-wave teams with full isolation. Teams created and destroyed per wave. Maximum parallelism and throughput.

## Key concepts

- **Tasks replace promises** — Instead of loop-based `<promise>` tags, phases complete via `TaskUpdate(status="completed")` with built-in dependency resolution.
- **Planning files as ground truth** — `task_plan.md`, `findings.md`, and `progress.md` persist across context windows, sessions, and teammate lifecycles. They are the durable state layer that survives ephemeral team operations.
- **Research as a first-class input** — Any field in the template can be marked `research: "..."` instead of `value: "..."`, deferring the decision to Phase 1 where research teammates investigate and the user confirms.
- **File ownership for parallel safety** — When teammates work in parallel, each phase explicitly defines which files it owns (exclusive edit) and which it reads (shared, no edit).

## How it compares to the loop-based approach

| Loop-Based (RALPH_LOOP_TEMPLATE) | Team-Based (AGENT_TEAM_TEMPLATE) |
|---|---|
| Single agent iterates in a loop | Multiple agents work concurrently |
| `<promise>` tags signal completion | `TaskUpdate(completed)` signals completion |
| Loop counter limits iterations | Task dependencies limit ordering |
| `/ralph-loop:ralph-loop "..."` | `TeamCreate` / `TaskCreate` / `Task(...)` |
| Teams are an optional add-on | Teams are the primary execution model |

## License

MIT
