# Claude Code Execution Templates

Parameterized templates for orchestrating [Claude Code](https://docs.anthropic.com/en/docs/claude-code) to build software projects end-to-end — from research and technology selection through implementation, deployment, and validation.

Two execution models, each with its own template and population guide:

## Execution Models

### Agent Team (`agent-team/`)

Uses Claude Code's native agent team features (`TeamCreate`, `TaskCreate`, `SendMessage`) as the primary execution model. Multiple agents work concurrently with task-based dependency resolution.

| File | Purpose |
|------|---------|
| [`agent-team/AGENT_TEAM_TEMPLATE.md`](agent-team/AGENT_TEAM_TEMPLATE.md) | The core template. Parameterized with `{{PLACEHOLDER}}` fields. Defines phases (Bootstrap, Assessment, Foundation, Features, Deployment, Validation) and three team topologies (solo, squad, swarm). |
| [`agent-team/AGENT_TEAM_POPULATE_GUIDE.md`](agent-team/AGENT_TEAM_POPULATE_GUIDE.md) | The interview prompt. 7-round interview to populate the template. Includes prerequisites and setup instructions. |

**Quick start:**
1. Copy `agent-team/AGENT_TEAM_TEMPLATE.md` and `agent-team/AGENT_TEAM_POPULATE_GUIDE.md` to your project root
2. Open Claude Code in your project directory
3. Copy the **Kickoff Prompt** from the populate guide (between `---START---` and `---END---`)
4. Paste it into Claude Code and answer 7 rounds of questions
5. Claude Code produces a populated `AGENT_TEAM_EXECUTION_GUIDE.md`

**Three topologies:**
- **Solo** — Sequential `Task()` calls. Cheapest, simplest.
- **Squad** — One persistent team for the project. Tasks with dependencies.
- **Swarm** — Per-wave teams. Maximum parallelism.

### Ralph Loop (`ralph-loop/`)

Uses the `ralph-loop` skill for iterative, loop-driven execution with `<promise>` tags for completion detection. A single agent iterates within each phase, with optional team support for parallel waves.

| File | Purpose |
|------|---------|
| [`ralph-loop/RALPH_LOOP_TEMPLATE.md`](ralph-loop/RALPH_LOOP_TEMPLATE.md) | The core template. Parameterized with `{{PLACEHOLDER}}` fields. Defines phases with `/ralph-loop:ralph-loop` commands, promise-based completion, and three execution modes (sequential, team, hybrid). |
| [`ralph-loop/RALPH_LOOP_POPULATE_GUIDE.md`](ralph-loop/RALPH_LOOP_POPULATE_GUIDE.md) | The interview prompt. 6-round interview to populate the template. Includes prerequisites and setup instructions. |

**Quick start:**
1. Copy `ralph-loop/RALPH_LOOP_TEMPLATE.md` and `ralph-loop/RALPH_LOOP_POPULATE_GUIDE.md` to your project root
2. Open Claude Code in your project directory
3. Copy the **Kickoff Prompt** from the populate guide (between `---START---` and `---END---`)
4. Paste it into Claude Code and answer 6 rounds of questions
5. Claude Code produces a populated `RALPH_LOOP_EXECUTION_GUIDE.md`

**Three execution modes:**
- **Sequential** — One ralph-loop at a time. User reviews between phases.
- **Team** — Parallel waves of teammates, each running their own ralph-loop.
- **Hybrid** — Sequential for dependent phases, parallel for independent ones.

## How the Two Models Compare

| | Ralph Loop | Agent Team |
|---|---|---|
| **Execution unit** | Single agent iterates in a loop | Multiple agents work concurrently |
| **Completion signal** | `<promise>` tags detected by the loop | `TaskUpdate(status="completed")` |
| **Iteration control** | `--max-iterations` per phase | Task dependencies limit ordering |
| **User control** | Review between each loop invocation | Review at topology level (solo/squad/swarm) |
| **Invocation** | `/ralph-loop:ralph-loop "..."` | `TeamCreate` / `TaskCreate` / `Task(...)` |
| **Team support** | Optional add-on (team/hybrid mode) | Primary execution model |
| **Best for** | Maximum user control, smaller projects, existing ralph-loop users | Maximum throughput, larger projects, native Claude Code workflows |

## Shared Concepts

Both models share these ideas:
- **Planning files** — `task_plan.md`, `findings.md`, `progress.md` persist state across phases
- **Research as input** — Any field accepts `research: "..."` to defer decisions to Phase 1
- **Memory MCP** — `Decision_` entities store resolved research across sessions
- **Agent selection** — Both templates include guides for choosing specialized subagent types
- **Phased execution** — Bootstrap → Assessment → Foundation → Features → Client Integration (optional) → Deployment → Validation

## License

MIT
