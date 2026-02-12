# Ralph Loop Execution Guide Template

> **How to use this template:**
> 1. Fill in the **Input Criteria** section below. Each field accepts EITHER a concrete value OR a research request.
> 2. Research requests get resolved during Phase 1 (Assessment) and stored in Memory as decisions.
> 3. Later phases reference those Memory decisions with `{{RESOLVED:field_name}}` instead of hardcoded values.
> 4. Copy the **Phase N** block once per major feature. Delete optional sections you don't need.
> 5. Choose an **Execution Mode**: `sequential` (one phase at a time), `team` (parallel waves using Claude Code agent teams), or `hybrid` (mix of both).
> 6. If using team/hybrid mode, define **waves** in `team_config` grouping independent phases for parallel execution.

---

## Input Criteria

Every field below accepts one of three input modes:

| Mode | Syntax | When to use |
|------|--------|-------------|
| **Concrete** | `value: "FastAPI"` | You already know the answer |
| **Research** | `research: "What backend framework best handles..."` | You want Phase 1 to decide |
| **Constrained research** | `research: "Compare X vs Y for..."` | You've narrowed it but want validation |

Fields marked `(required)` must have at least a research request. Fields marked `(optional)` can be omitted entirely.

```yaml
# ╔══════════════════════════════════════════════════════════════╗
# ║  PROJECT IDENTITY (required)                                 ║
# ╚══════════════════════════════════════════════════════════════╝
project_name: "{{PROJECT_NAME}}"               # Always concrete
project_slug: "{{PROJECT_SLUG}}"               # Always concrete (lowercase, hyphens)
prompt_file: "{{PROMPT_FILE}}"                 # The prompt doc this guide executes
goal_summary: "{{GOAL_SUMMARY}}"               # 1-2 sentence goal. Always concrete.

# ╔══════════════════════════════════════════════════════════════╗
# ║  ARCHITECTURE (concrete OR research per field)               ║
# ╚══════════════════════════════════════════════════════════════╝
tech_stack:
  backend_framework:                           # Pick ONE mode per field:
    value: "FastAPI"                           #   Concrete: you know
    # -- OR --
    # research: "What Python backend framework is best for real-time microservices with WebSocket support?"
    # constraints: ["Must be Python", "Must support async"]  # optional narrowing

  backend_language:
    value: "Python"
    # research: "What language best balances developer velocity and production performance for a chat platform?"

  frontend_framework:
    value: "React"
    # research: "Compare React vs Vue vs Svelte for a real-time chat UI with complex state management"

  database:
    value: "PostgreSQL"
    # research: "What database handles mixed read-heavy chat history and write-heavy message ingestion?"
    # constraints: ["Must be self-hostable", "Must support full-text search"]

  cache:
    value: "Redis"
    # research: "Do we need a cache layer, and if so what fits best for pub/sub + session storage?"

  deployment:
    value: "Docker Compose"
    # research: "What deployment approach works best for self-hosted, single-server, 50-100 users?"
    # constraints: ["No Kubernetes", "Must be simple for non-DevOps users"]

  client_type:
    value: "Electron desktop"
    # research: "What client platform reaches the most users for a self-hosted chat app?"
    # constraints: ["Must support WebRTC", "Must work on Windows/Mac/Linux"]

# Service list: concrete if you have a codebase, research if greenfield
services:
  value:                                       # Concrete: list known services
    - "auth"
    - "user"
    - "server"
    - "channel"
    - "message"
    - "file"
    - "webrtc"
    - "websocket"
  # -- OR --
  # research: "What service boundaries make sense for a chat platform with text, voice, and file sharing?"
  # constraints: ["Minimize inter-service calls", "Each service owns its own DB tables"]

service_base_path: "backend/services"          # Where services live in the repo
service_entry_file: "main.py"                  # Main file per service

# ╔══════════════════════════════════════════════════════════════╗
# ║  DEPLOYMENT TARGET (required)                                ║
# ╚══════════════════════════════════════════════════════════════╝
hosting:
  value: "self-hosted"
  # research: "Should this be self-hosted, cloud-managed, or hybrid for a small team's internal chat?"

target_users:
  value: "50-100 concurrent"
  # research: "What user scale should we design for given a single-server Docker Compose deployment?"

geo_distribution:
  value: "US geo-distributed"
  # research: "What geographic distribution pattern matters for a US-based team chat with voice?"

# ╔══════════════════════════════════════════════════════════════╗
# ║  FEATURES (required - list what needs implementation)        ║
# ╚══════════════════════════════════════════════════════════════╝
# Each feature becomes its own implementation phase.
# Use 'research' within a feature to investigate HOW to implement it.
features:
  - name: "Real-Time Messaging"
    category: "websocket"
    critical_path: false
    approach:
      value: "WebSocket gateway with Redis pub/sub"
      # research: "What real-time messaging pattern works best for horizontal scaling without Kubernetes?"

  - name: "Voice Communication"
    category: "webrtc"
    critical_path: true                        # The riskiest/hardest feature
    approach:
      # research: "Compare mediasoup vs LiveKit vs Janus for a self-hosted SFU supporting 20 concurrent voice users"
      constraints: ["Must work behind NAT", "Must be Docker-deployable", "Sub-200ms latency"]

  - name: "File Uploads"
    category: "storage"
    critical_path: false
    approach:
      value: "MinIO (S3-compatible) with presigned URLs"

# ╔══════════════════════════════════════════════════════════════╗
# ║  EXECUTION MODE (required)                                   ║
# ╚══════════════════════════════════════════════════════════════╝
# Choose how phases are executed:
#   "sequential" - One ralph-loop at a time (default, lowest token cost)
#   "team"       - Agent team with parallel waves (higher throughput, higher token cost)
#   "hybrid"     - Sequential for dependent phases, team for parallel-eligible phases

execution_mode: "sequential"                     # "sequential" | "team" | "hybrid"

# Team configuration (only used when execution_mode is "team" or "hybrid")
team_config:
  team_name: "{{PROJECT_SLUG}}-build"            # Team identifier
  delegate_lead: true                            # true = lead only coordinates, never implements
  require_plan_approval: true                    # Teammates must get plan approved before coding
  default_teammate_model: "sonnet"               # "sonnet" | "opus" | "haiku" per teammate

  # Waves: groups of phases that run in parallel.
  # Phases within a wave have NO dependencies on each other.
  # Waves run sequentially (wave 0 finishes before wave 1 starts).
  waves:
    - wave: 0
      mode: "solo"                               # Lead runs these directly
      phases: ["step0", "phase1"]
    - wave: 1
      mode: "solo"
      phases: ["phase2"]
    - wave: 2
      mode: "parallel"                           # Spawn teammates for these
      phases: ["phase3", "phase4", "phase5"]     # Must match feature names
      teammates:
        - name: "{{FEATURE_1_SLUG}}-dev"
          phase: "phase3"
          subagent_type: "general-purpose"        # or specialized type from Agent Selection Guide
        - name: "{{FEATURE_2_SLUG}}-dev"
          phase: "phase4"
          subagent_type: "general-purpose"
        - name: "{{FEATURE_3_SLUG}}-dev"
          phase: "phase5"
          subagent_type: "general-purpose"
    - wave: 3
      mode: "solo"
      phases: ["phaseY"]
    - wave: 4
      mode: "parallel"
      phases: ["phaseD", "phaseF-prep"]          # Deployment + test prep can overlap
      teammates:
        - name: "deploy-engineer"
          phase: "phaseD"
          subagent_type: "general-purpose"
        - name: "test-engineer"
          phase: "phaseF-prep"
          subagent_type: "general-purpose"
    - wave: 5
      mode: "solo"
      phases: ["phaseF"]                         # Final validation is always sequential

# ╔══════════════════════════════════════════════════════════════╗
# ║  SUCCESS CRITERIA (required)                                 ║
# ╚══════════════════════════════════════════════════════════════╝
# Concrete items that the final phase validates. These should be specific and testable.
# Use 'research' if you need help defining what "good" looks like.
success_criteria:
  functional:
    - value: "50+ concurrent users connect from different locations"
    - value: "Text messaging with real-time delivery"
    - value: "Voice channels with clear audio across NAT"
    # - research: "What functional acceptance criteria are standard for production chat platforms?"

  infrastructure:
    - value: "SSL/TLS on all services"
    - value: "Database backups automated"
    # - research: "What infrastructure criteria matter for a self-hosted deployment?"

  deployment:
    - value: "Single command starts everything"
    - value: ".env.example documented"

  performance:
    - value: "Voice latency < 200ms coast-to-coast"
    - value: "Message delivery < 500ms"
    # - research: "What performance benchmarks are reasonable for 50-100 user self-hosted chat?"
```

---

## How Research Requests Flow Through the Template

```
Input Criteria                    Phase 1 Assessment              Phases 2+
┌─────────────────┐              ┌──────────────────┐            ┌──────────────────┐
│ backend_framework│              │                  │            │                  │
│   research: "X?" │─────────────│  Octagon/PAL/    │            │ Read Memory:     │
│   constraints:   │   feeds     │  Brave research  │  stores    │ "Decision_       │
│   ["Must be Y"]  │   into      │  resolves to     │──────────│  backend_fw"     │
│                  │              │  concrete choice │  Memory   │ → Use resolved   │
└─────────────────┘              │  + rationale     │  entity   │   value           │
                                 └──────────────────┘            └──────────────────┘

┌─────────────────┐              ┌──────────────────┐            ┌──────────────────┐
│ backend_framework│              │                  │            │                  │
│   value: "FastAPI"│─────────────│  Skips research, │            │ Uses "FastAPI"   │
│                  │   passes    │  validates only   │  stores   │ directly         │
│                  │   through   │                  │──────────│                  │
└─────────────────┘              └──────────────────┘            └──────────────────┘
```

---

## Team Execution Architecture

> **Skip this section if `execution_mode` is `"sequential"`.** When using team or hybrid mode,
> this section explains how phases map to teammates and how the team lead orchestrates execution.

### How It Works

```
                    ┌─────────────────────────────────────────────────┐
                    │               TEAM LEAD (you)                   │
                    │  • Creates team via TeamCreate                   │
                    │  • Creates tasks via TaskCreate                  │
                    │  • Spawns teammates via Task tool                │
                    │  • Uses delegate mode (coordinates, never codes) │
                    │  • Sends shutdown_request when wave completes    │
                    └────────────┬────────────────────────────────────┘
                                 │
            ┌────────────────────┼────────────────────┐
            │                    │                    │
    ┌───────▼───────┐   ┌───────▼───────┐   ┌───────▼───────┐
    │  Teammate A   │   │  Teammate B   │   │  Teammate C   │
    │  Phase 3      │   │  Phase 4      │   │  Phase 5      │
    │  (own context)│   │  (own context)│   │  (own context)│
    │               │   │               │   │               │
    │  Runs its own │   │  Runs its own │   │  Runs its own │
    │  ralph-loop   │   │  ralph-loop   │   │  ralph-loop   │
    │  internally   │   │  internally   │   │  internally   │
    └───────────────┘   └───────────────┘   └───────────────┘
            │                    │                    │
            └────────────────────┼────────────────────┘
                                 │
                    ┌────────────▼────────────────────┐
                    │        Shared Task List          │
                    │  ~/.claude/tasks/{team-name}/    │
                    │                                  │
                    │  Task 1: Phase 3 [in_progress]   │
                    │  Task 2: Phase 4 [in_progress]   │
                    │  Task 3: Phase 5 [pending]       │
                    │    blockedBy: [Task 1]           │
                    └─────────────────────────────────┘
```

### Wave Execution Model

Phases are grouped into **waves**. Phases within a wave run in parallel as separate teammates.
Waves run sequentially — wave N+1 starts only after all teammates in wave N finish.

```
SEQUENTIAL (execution_mode: "sequential")        TEAM (execution_mode: "team")
─────────────────────────────────────            ──────────────────────────────────
Step 0 ──→ Phase 1 ──→ Phase 2                   Wave 0: Step 0 → Phase 1 (solo)
  ──→ Phase 3 ──→ Phase 4 ──→ Phase 5            Wave 1: Phase 2 (solo)
  ──→ Phase Y ──→ Phase D ──→ Phase F            Wave 2: Phase 3 ║ Phase 4 ║ Phase 5
                                                  Wave 3: Phase Y (solo)
Total: serial execution of all phases             Wave 4: Phase D ║ Phase F-prep
                                                  Wave 5: Phase F (solo)

                                                  Phases 3/4/5 run SIMULTANEOUSLY
```

### Teammate Lifecycle

```
1. TeamCreate(team_name="{{PROJECT_SLUG}}-wave-{{N}}")
   └─ Creates team + shared task list

2. TaskCreate(...) for each phase in the wave
   └─ Establishes task dependencies via addBlockedBy

3. Task(subagent_type=..., team_name=..., name=..., prompt=...)
   └─ Spawns each teammate with its phase instructions
   └─ Teammate loads CLAUDE.md + MCP servers automatically
   └─ Teammate receives phase prompt as spawn context

4. Teammates work independently:
   └─ Each runs ralph-loop (or direct implementation) for its phase
   └─ Reads Memory for Decision_ entities from Phase 1
   └─ Writes Memory for its own decisions/checkpoints
   └─ Marks its task as completed via TaskUpdate

5. SendMessage(type="shutdown_request", recipient="teammate-name")
   └─ Graceful shutdown after wave completes

6. TeamDelete()
   └─ Clean up team resources between waves (or at end)
```

### When to Use Teams vs Sequential

| Scenario | Recommended Mode | Why |
|----------|-----------------|-----|
| 0-2 features, tight dependencies | `sequential` | Overhead exceeds benefit |
| 3+ features, some parallelizable | `hybrid` | Parallel where possible, sequential otherwise |
| 5+ features, clear boundaries | `team` | Maximum throughput |
| Research-heavy (8+ open questions) | `team` for Phase 1 | Parallel research with competing hypotheses |
| Single developer reviewing output | `sequential` | Easier to follow and review |
| CI/CD or automated execution | `team` | Maximize throughput, minimal human review needed |

---

# Ralph Loop Execution Guide for {{PROMPT_FILE}}

**Purpose**: {{GOAL_SUMMARY}}

**Execution Order**: Step 0 -> Phase 1{{PHASE_ARROW_CHAIN}}

**State Continuity**: Each phase reads accumulated state from `task_plan.md`, `findings.md`, `progress.md`, and Memory MCP entities written by prior phases.

**Execution Mode**: {{EXECUTION_MODE}} {{IF_TEAM_MODE}}({{PARALLEL_WAVE_COUNT}} parallel waves, max {{MAX_CONCURRENT}} concurrent teammates){{/IF_TEAM_MODE}}

**Control Flow**: {{IF_SEQUENTIAL}}After each ralph-loop completes (promise detected or max iterations), review output before starting the next phase.{{/IF_SEQUENTIAL}}{{IF_TEAM_MODE}}The team lead orchestrates waves of parallel teammates. Each teammate runs its own ralph-loop internally. Between waves, the lead verifies all promises, resolves conflicts, and proceeds to the next wave.{{/IF_TEAM_MODE}} Use `/ralph-loop:cancel-ralph` if a loop stalls.

**Open Research**: {{COUNT}} fields require research resolution in Phase 1 before implementation can begin.

---

## Step 0: Bootstrap

**Purpose**: Initialize all tools, register project, load prior context, set up planning files.
**Depends on**: Nothing (this is first)
**Produces**: Serena onboarded, Memory initialized, planning files created, checkpoint saved

```
/ralph-loop:ralph-loop "Execute {{PROMPT_FILE}} Step 0: Bootstrap.

TASKS:
1. mcp__serena__check_onboarding_performed() - check if project is registered
2. mcp__serena__onboarding() - register project if not already done
3. mcp__serena__initial_instructions() - read Serena operating manual
4. mcp__memory__read_graph() - check for any prior context from previous sessions
5. mcp__memory__search_nodes('{{PROJECT_SLUG}}') - find related prior work
6. /planning-with-files:plan '{{PROJECT_NAME}} - {{GOAL_SUMMARY}}'
7. Verify task_plan.md, findings.md, progress.md are created
8. mcp__memory__create_entities([{name: 'Project_{{PROJECT_SLUG}}_Start', entityType: 'milestone', observations: ['{{GOAL_SUMMARY}}', 'Target: {{HOSTING}}, {{TARGET_USERS}}, {{GEO_DISTRIBUTION}}', 'Open research questions: {{RESEARCH_FIELD_LIST}}', 'Execution mode: {{EXECUTION_MODE}}']}])
{{IF_TEAM_MODE}}
9. TEAM SETUP (only if execution_mode is 'team' or 'hybrid'):
   - Store team config in Memory: mcp__memory__create_entities([{name: 'TeamConfig_{{PROJECT_SLUG}}', entityType: 'team_config', observations: ['team_name: {{TEAM_NAME}}', 'waves: {{WAVE_DEFINITIONS}}', 'delegate_lead: {{DELEGATE_LEAD}}', 'require_plan_approval: {{REQUIRE_PLAN_APPROVAL}}']}])
   - Note: TeamCreate is called at the start of each parallel wave, NOT here
{{/IF_TEAM_MODE}}
10. /checkpoint:create 'pre-assessment-bootstrap'

Output <promise>BOOTSTRAP_COMPLETE</promise> when Serena is onboarded, Memory is initialized, planning files exist, {{IF_TEAM_MODE}}team config is stored in Memory, {{/IF_TEAM_MODE}}and checkpoint is saved." --max-iterations {{IF_TEAM_MODE}}7{{/IF_TEAM_MODE}}{{IF_SEQUENTIAL}}5{{/IF_SEQUENTIAL}} --completion-promise "BOOTSTRAP_COMPLETE"
```

---

## Phase 1: Assessment + Technology Selection

**Purpose**: Full gap analysis of the codebase AND resolution of all open research questions into concrete technology decisions.
**Depends on**: Step 0 (Serena onboarded, planning files exist)
**Produces**: Populated `findings.md`, detailed `task_plan.md`, Memory entities with assessment results AND all technology decisions resolved

> **Key difference from a pure-assessment phase:** This phase has a dedicated "RESOLVE OPEN QUESTIONS"
> section that turns every `research:` field from Input Criteria into a concrete Memory decision entity.
> Later phases read these decisions from Memory instead of using hardcoded values.

```
/ralph-loop:ralph-loop "Execute {{PROMPT_FILE}} Phase 1: Assessment + Technology Selection.

Read task_plan.md and progress.md for current state.

PART A: RESOLVE OPEN RESEARCH QUESTIONS
========================================
The following fields need research-based decisions. For each one:
  (a) Run parallel research using Octagon + Brave + PAL
  (b) Synthesize into a recommendation with rationale
  (c) AskUserQuestion to confirm or override the recommendation
  (d) Store as Memory entity: Decision_<field_name>

{{BLOCK:RESEARCH_FIELD}}
OPEN QUESTION: {{FIELD_NAME}}
  Request: '{{RESEARCH_REQUEST}}'
  Constraints: {{CONSTRAINTS}}
  Research actions:
  - mcp__octagon-deep-research-mcp__octagon-deep-research-agent: '{{RESEARCH_REQUEST}} Consider: {{CONSTRAINTS}}. Evaluate in context of {{GOAL_SUMMARY}}. Year: 2026.'
  - mcp__brave-search__brave_web_search: '{{BRAVE_SEARCH_VARIANT}} 2026'
  - mcp__pal__thinkdeep: 'For a project that {{GOAL_SUMMARY}}, evaluate: {{RESEARCH_REQUEST}}'
  After synthesis:
  - AskUserQuestion: 'Based on research, I recommend {{RECOMMENDATION}} because {{RATIONALE}}. Options: [recommended] / [alternative_1] / [alternative_2] / [custom]'
  - mcp__memory__create_entities([{name: 'Decision_{{FIELD_NAME}}', entityType: 'technology_decision', observations: ['Question: {{RESEARCH_REQUEST}}', 'Decision: <chosen_value>', 'Rationale: <why>', 'Alternatives considered: <list>', 'Constraints applied: {{CONSTRAINTS}}']}])
{{/BLOCK:RESEARCH_FIELD}}

PART B: CODEBASE GAP ANALYSIS (parallel with Part A where possible)
====================================================================

1. GAP ANALYSIS (Serena):
   - mcp__serena__search_for_pattern('TODO|FIXME|mock|stub|placeholder|NotImplemented') across all {{SERVICE_BASE_PATH}}/
   {{IF_FRONTEND}}- mcp__serena__search_for_pattern('TODO|FIXME|mock|stub|placeholder') across {{FRONTEND_PATH}}/{{/IF_FRONTEND}}
   - mcp__serena__list_dir('{{SERVICE_BASE_PATH}}', recursive=True)
   - mcp__serena__get_symbols_overview() on every {{SERVICE_ENTRY_FILE}} in {{SERVICE_BASE_PATH}}/*/
   - Document every gap found in findings.md

2. ARCHITECTURE EVALUATION (PAL + Octagon):
   - mcp__pal__thinkdeep: 'Evaluate the {{PROJECT_NAME}} architecture using the resolved technology decisions from Memory. Target: {{HOSTING}}, {{TARGET_USERS}}, {{GEO_DISTRIBUTION}}.'
   - mcp__pal__challenge: '{{ARCHITECTURE_CHALLENGE_QUESTION}}'
   - Store results in Memory

3. SECURITY AUDIT (Agent + PAL):
   - Task(subagent_type='comprehensive-review:security-auditor') on {{SERVICE_BASE_PATH}}/
   - mcp__pal__codereview focusing on security vulnerabilities
   - Document findings in findings.md

4. DATABASE ANALYSIS (Agent):
   - Task(subagent_type='database-cloud-optimization:database-optimizer') on database schemas

5. WEB RESEARCH (Brave):
   {{BLOCK:BRAVE_SEARCH}}
   - mcp__brave-search__brave_web_search: '{{SEARCH_QUERY}}'
   {{/BLOCK:BRAVE_SEARCH}}

6. MODEL SELECTION (PAL):
   - mcp__pal__listmodels() - verify best available models for subsequent phases

PART C: SYNTHESIZE
===================
- Verify ALL research questions from Part A have Decision_ entities in Memory
- Compile gap analysis + architecture evaluation + security findings into findings.md
- Create task_plan.md with per-phase breakdown referencing resolved decisions
- mcp__pal__consensus on overall architecture approach using resolved tech stack
- Update progress.md with Phase 1 completion and all decisions made
- /checkpoint:create 'phase1-assessment-complete'

Output <promise>ASSESSMENT_COMPLETE</promise> when:
  1. ALL research fields have Decision_ entities in Memory
  2. findings.md has comprehensive gap analysis
  3. task_plan.md has detailed phase breakdown using resolved technology choices
  4. User has confirmed or overridden every technology recommendation" --max-iterations {{ASSESSMENT_MAX_ITER}} --completion-promise "ASSESSMENT_COMPLETE"
```

---

## Team Lead Orchestration Script (Team/Hybrid Mode Only)

> **Skip this section if `execution_mode` is `"sequential"`.** In sequential mode, run each phase
> as an individual ralph-loop command in order. In team/hybrid mode, use this orchestration script
> to manage waves of parallel teammates.
>
> **When to use:** After Phase 1 completes and all research questions are resolved. The lead
> runs this script to execute all remaining phases using the wave definitions from `team_config`.

```
/ralph-loop:ralph-loop "TEAM LEAD ORCHESTRATION for {{PROMPT_FILE}}.

You are the TEAM LEAD. You coordinate teammates. {{IF_DELEGATE_LEAD}}You are in DELEGATE MODE — you do NOT write code or make file changes yourself. You only:
- Create teams (TeamCreate)
- Create and assign tasks (TaskCreate, TaskUpdate)
- Spawn teammates (Task tool with team_name)
- Send messages (SendMessage)
- Monitor progress (TaskList)
- Shut down teammates (SendMessage type: shutdown_request)
- Clean up teams (TeamDelete){{/IF_DELEGATE_LEAD}}

PREREQUISITES (verify before proceeding):
- BOOTSTRAP_COMPLETE promise fulfilled
- ASSESSMENT_COMPLETE promise fulfilled
- All Decision_ entities exist in Memory: mcp__memory__search_nodes('Decision_')
- task_plan.md has per-phase breakdown
- findings.md has gap analysis

{{BLOCK:WAVE}}
═══════════════════════════════════════════════════
WAVE {{WAVE_NUM}}: {{WAVE_DESCRIPTION}}
═══════════════════════════════════════════════════
{{IF_WAVE_SOLO}}
Run Phase {{PHASE_ID}} directly as a single ralph-loop (see phase section below).
Wait for {{PHASE_PROMISE}} before proceeding to next wave.
{{/IF_WAVE_SOLO}}
{{IF_WAVE_PARALLEL}}
1. CREATE TEAM:
   TeamCreate(team_name='{{PROJECT_SLUG}}-wave-{{WAVE_NUM}}', description='Wave {{WAVE_NUM}}: {{WAVE_DESCRIPTION}}')

2. CREATE TASKS WITH DEPENDENCIES:
   {{BLOCK:WAVE_TASK}}
   TaskCreate(
     subject='Phase {{N}}: {{PHASE_TITLE}}',
     description='{{PHASE_PURPOSE}}. Promise: {{PHASE_PROMISE}}. Max iterations: {{PHASE_MAX_ITER}}.',
     activeForm='Executing Phase {{N}}: {{PHASE_TITLE}}'
   )
   {{IF_TASK_DEPENDENCY}}
   TaskUpdate(taskId='{{TASK_ID}}', addBlockedBy=['{{BLOCKING_TASK_ID}}'])
   {{/IF_TASK_DEPENDENCY}}
   {{/BLOCK:WAVE_TASK}}

3. SPAWN TEAMMATES:
   {{BLOCK:TEAMMATE_SPAWN}}
   Task(
     subagent_type='{{TEAMMATE_SUBAGENT_TYPE}}',
     team_name='{{PROJECT_SLUG}}-wave-{{WAVE_NUM}}',
     name='{{TEAMMATE_NAME}}',
     {{IF_REQUIRE_PLAN_APPROVAL}}mode='plan',{{/IF_REQUIRE_PLAN_APPROVAL}}
     {{IF_TEAMMATE_MODEL}}model='{{TEAMMATE_MODEL}}',{{/IF_TEAMMATE_MODEL}}
     prompt='You are teammate {{TEAMMATE_NAME}} on team {{PROJECT_SLUG}}-wave-{{WAVE_NUM}}.

YOUR TASK: Execute Phase {{N}}: {{PHASE_TITLE}}.
{{PHASE_PURPOSE}}

CONTEXT LOADING:
1. Read task_plan.md, findings.md, progress.md for project state
2. mcp__memory__search_nodes(\"Decision_\") — load ALL resolved technology decisions
3. mcp__memory__search_nodes(\"{{PHASE_MEMORY_TAG}}\") — load phase-specific context
4. Check TaskList for your assigned task and any dependencies

PHASE INSTRUCTIONS:
{{PHASE_INSTRUCTIONS_INLINE}}

COMPLETION:
- When done, mark your task completed: TaskUpdate(taskId=..., status=\"completed\")
- Store results: mcp__memory__create_entities for phase decisions/artifacts
- Update progress.md with your phase results
- /checkpoint:create \"phase{{N}}-{{PHASE_SLUG}}-complete\"
- Send completion message to lead: SendMessage(type=\"message\", recipient=\"team-lead\", content=\"Phase {{N}} complete. {{PHASE_PROMISE}} fulfilled.\", summary=\"Phase {{N}} done\")
'
   )
   {{/BLOCK:TEAMMATE_SPAWN}}

4. MONITOR PROGRESS:
   - TaskList() periodically to check task status
   - Messages from teammates arrive automatically
   - If a teammate goes idle but task is not complete, send them a nudge:
     SendMessage(type='message', recipient='{{TEAMMATE_NAME}}', content='Your task is not yet complete. Please continue.', summary='Continue working')
   - If a teammate is stuck, consider:
     a. Sending guidance via SendMessage
     b. Spawning a helper teammate for the blocking issue

5. WAIT FOR ALL PROMISES: {{WAVE_PROMISE_LIST}}

6. SHUTDOWN WAVE:
   {{BLOCK:TEAMMATE_SHUTDOWN}}
   SendMessage(type='shutdown_request', recipient='{{TEAMMATE_NAME}}', content='Wave {{WAVE_NUM}} complete. Shutting down.')
   {{/BLOCK:TEAMMATE_SHUTDOWN}}
   - Wait for all shutdown confirmations
   - TeamDelete()
   - /checkpoint:create 'wave-{{WAVE_NUM}}-complete'
{{/IF_WAVE_PARALLEL}}
{{/BLOCK:WAVE}}

FINAL ORCHESTRATION:
- Verify ALL phase promises are fulfilled
- mcp__memory__read_graph() — confirm complete decision trail
- Proceed to Phase F (Final Validation) if not already run as part of a wave

Output <promise>ORCHESTRATION_COMPLETE</promise> when all waves have finished, all teams are cleaned up, and all phase promises are fulfilled." --max-iterations {{ORCHESTRATION_MAX_ITER}} --completion-promise "ORCHESTRATION_COMPLETE"
```

### Teammate Spawn Patterns

For different phase types, use these specialized teammate configurations:

| Phase Type | Recommended `subagent_type` | Notes |
|------------|---------------------------|-------|
| Backend feature | `general-purpose` | Full tool access for code changes |
| Frontend feature | `general-purpose` | Needs Edit/Write for component work |
| Security audit | `comprehensive-review:security-auditor` | Read-heavy, minimal edits |
| Performance testing | `full-stack-orchestration:performance-engineer` | Needs Bash for load tests |
| Deployment | `general-purpose` | Docker/config file editing |
| Research-only | `Explore` | Read-only, cheaper for investigation |

### Competing Hypotheses Pattern (for Critical Path)

For critical-path features, spawn multiple teammates to investigate different approaches simultaneously:

```
# Instead of one teammate for Phase 3 (critical path), spawn 3 investigators:
Task(
  subagent_type='Explore',
  team_name='{{PROJECT_SLUG}}-wave-2',
  name='approach-a-investigator',
  prompt='Investigate approach A for {{CRITICAL_FEATURE}}: {{APPROACH_A_DESCRIPTION}}.
  Research viability, document trade-offs, store findings in Memory.
  Send findings to team lead when done.'
)
Task(
  subagent_type='Explore',
  team_name='{{PROJECT_SLUG}}-wave-2',
  name='approach-b-investigator',
  prompt='Investigate approach B for {{CRITICAL_FEATURE}}: {{APPROACH_B_DESCRIPTION}}.
  Research viability, document trade-offs, store findings in Memory.
  Send findings to team lead when done.'
)
Task(
  subagent_type='general-purpose',
  team_name='{{PROJECT_SLUG}}-wave-2',
  name='challenger',
  prompt='You are the devils advocate. Read the findings from approach-a-investigator
  and approach-b-investigator. Challenge both. Find weaknesses. Recommend which
  approach to proceed with and why.'
)
```

After the challenge round, the lead picks the winning approach and spawns an implementation teammate.

---

## Phase 2: {{PHASE_2_TITLE}}

> **Template note:** Phase 2 is typically the foundational implementation phase (core APIs, auth, data layer).
> All `{{RESOLVED:*}}` references pull from Memory Decision_ entities created in Phase 1.

**Purpose**: {{PHASE_2_PURPOSE}}
**Depends on**: Phase 1 (all technology decisions resolved, gap analysis complete)
**Produces**: {{PHASE_2_PRODUCES}}

```
/ralph-loop:ralph-loop "Execute {{PROMPT_FILE}} Phase 2: {{PHASE_2_TITLE}}.

Read task_plan.md, findings.md, and progress.md for context and known gaps.
Read Memory for resolved decisions: mcp__memory__search_nodes('Decision_')
Read Memory for Phase 1 results: mcp__memory__search_nodes('assessment')

Use the resolved technology decisions from Memory for all implementation choices:
- Backend framework: read Decision_backend_framework
- Database: read Decision_database
- Cache: read Decision_cache
- (etc. for any field that was a research question)

FOR EACH SERVICE ({{SERVICES_LIST}} -- or read Decision_services if services were researched):

1. READ current state:
   - mcp__serena__get_symbols_overview('{{SERVICE_BASE_PATH}}/<service>/{{SERVICE_ENTRY_FILE}}')
   - Identify all TODO/mock/placeholder code

2. RESEARCH implementation patterns:
   - mcp__pal__apilookup: '{{RESOLVED:backend_framework}} <specific_pattern> production implementation'

3. IMPLEMENT with TDD:
   - Task(subagent_type='backend-development:tdd-orchestrator') - write tests first
   - mcp__serena__replace_symbol_body() - replace mocks with real implementations
   - mcp__serena__insert_after_symbol() - add missing endpoints
   {{BLOCK:IMPLEMENTATION_DETAIL}}
   - {{IMPLEMENTATION_STEP}}
   {{/BLOCK:IMPLEMENTATION_DETAIL}}

4. REVIEW each service:
   - mcp__pal__codereview: production readiness, security, error handling
   - Task(subagent_type='{{SECURITY_AGENT}}') for security review

5. CHECKPOINT after each service:
   - mcp__memory__create_entities for service completion state
   - Update progress.md
   - /checkpoint:create '<service>-complete'

6. INTEGRATION:
   - Verify inter-service communication works
   - Test with mcp__fetch__fetch on health endpoints
   - Task(subagent_type='full-stack-orchestration:test-automator') for integration tests

Output <promise>{{PHASE_2_PROMISE}}</promise> when {{PHASE_2_DONE_CRITERIA}}." --max-iterations {{PHASE_2_MAX_ITER}} --completion-promise "{{PHASE_2_PROMISE}}"
```

---

## Phase N: {{PHASE_N_TITLE}} (Repeatable Feature Phase)

> **Template note:** Copy this section for each feature from your `features` list.
> If the feature's `approach` was a research question, Phase 1 resolved it -- read from Memory.
>
> **Team mode:** In team/hybrid mode, this phase's ralph-loop command becomes the `prompt` parameter
> when spawning a teammate. The lead does NOT run this ralph-loop directly — instead, it wraps
> these instructions into a `Task(...)` call. See "Team Lead Orchestration Script" above.

**Purpose**: {{PHASE_N_PURPOSE}}
**Depends on**: {{PHASE_N_DEPENDS}}
**Produces**: {{PHASE_N_PRODUCES}}

```
/ralph-loop:ralph-loop "Execute {{PROMPT_FILE}} Phase N: {{PHASE_N_TITLE}}.{{IF_CRITICAL}} THIS IS THE CRITICAL PATH.{{/IF_CRITICAL}}

Read task_plan.md, findings.md, progress.md for context.
Read Memory: mcp__memory__search_nodes('Decision_') for all resolved technology decisions.
Read Memory: mcp__memory__search_nodes('{{PHASE_N_MEMORY_TAG}}') for feature-specific context.

{{IF_APPROACH_WAS_RESEARCHED}}
Read the implementation approach from Memory: mcp__memory__open_nodes(['Decision_{{FEATURE_CATEGORY}}_approach'])
Use the resolved approach for all implementation decisions in this phase.
{{/IF_APPROACH_WAS_RESEARCHED}}

{{IF_NEEDS_ADDITIONAL_RESEARCH}}DEEP RESEARCH (parallel):
{{BLOCK:OCTAGON_RESEARCH}}
- mcp__octagon-deep-research-mcp__octagon-deep-research-agent: '{{RESEARCH_PROMPT}}'
{{/BLOCK:OCTAGON_RESEARCH}}
{{BLOCK:API_LOOKUP}}
- mcp__pal__apilookup: '{{API_LOOKUP_QUERY}}'
{{/BLOCK:API_LOOKUP}}
{{BLOCK:BRAVE_SEARCH}}
- mcp__brave-search__brave_web_search: '{{SEARCH_QUERY}}'
{{/BLOCK:BRAVE_SEARCH}}
{{/IF_NEEDS_ADDITIONAL_RESEARCH}}

IMPLEMENTATION:

{{BLOCK:IMPLEMENTATION_SECTION}}
{{SECTION_NUMBER}}. {{SECTION_TITLE}} ({{SECTION_PATH}}):
   - mcp__serena__get_symbols_overview on relevant files
   {{BLOCK:SECTION_STEP}}
   - {{STEP_DESCRIPTION}}
   {{/BLOCK:SECTION_STEP}}
   - mcp__serena__replace_symbol_body / insert_after_symbol for implementation
{{/BLOCK:IMPLEMENTATION_SECTION}}

TESTING:
   - Task(subagent_type='backend-development:tdd-orchestrator') for feature tests
   {{BLOCK:TEST_STEP}}
   - {{TEST_DESCRIPTION}}
   {{/BLOCK:TEST_STEP}}
   - mcp__pal__codereview

CHECKPOINT:
   - mcp__memory__create_entities for {{PHASE_N_MEMORY_TAG}} architecture decisions
   - Update progress.md
   - /checkpoint:create 'phase{{N}}-{{PHASE_N_SLUG}}-complete'

Output <promise>{{PHASE_N_PROMISE}}</promise> when {{PHASE_N_DONE_CRITERIA}}." --max-iterations {{PHASE_N_MAX_ITER}} --completion-promise "{{PHASE_N_PROMISE}}"
```

---

## Phase Y: Client Integration (optional)

> Include if your project has a separate client. Delete if not applicable.
> If `client_type` was a research question, this phase reads the resolved decision from Memory.

**Purpose**: Connect the client to real backend APIs, replacing all mock data.
**Depends on**: All feature phases
**Produces**: Client fully connected with all features working

```
/ralph-loop:ralph-loop "Execute {{PROMPT_FILE}} Phase Y: Client Integration.

Read task_plan.md, findings.md, progress.md for context.
Read Memory: mcp__memory__search_nodes('Decision_') for resolved technology decisions.
Read Memory: mcp__memory__search_nodes('{{CLIENT_MEMORY_TAG}}') for client context.

IMPLEMENTATION:

1. ANALYZE CURRENT STATE:
   - mcp__serena__list_dir('{{CLIENT_SRC_PATH}}', recursive=True)
   - mcp__serena__get_symbols_overview on key source files
   - mcp__serena__search_for_pattern('mock|fake|dummy|placeholder|TODO') in {{CLIENT_SRC_PATH}}/

2. API CLIENT LAYER:
   - Replace mock API calls with real backend calls
   - Implement: base URL configuration, auth token management, error handling
   - Task(subagent_type='{{CLIENT_AGENT}}') for component integration

{{BLOCK:CLIENT_FEATURE}}
{{FEATURE_NUMBER}}. {{CLIENT_FEATURE_TITLE}}:
   {{BLOCK:CLIENT_FEATURE_STEP}}
   - {{CLIENT_STEP}}
   {{/BLOCK:CLIENT_FEATURE_STEP}}
{{/BLOCK:CLIENT_FEATURE}}

TESTING:
   - Task(subagent_type='{{CLIENT_SECURITY_AGENT}}') for client security audit
   - mcp__pal__codereview on integration code
   - Verify zero mock data remains: mcp__serena__search_for_pattern('mock|fake|dummy') should return nothing

CHECKPOINT:
    - mcp__memory__create_entities for client integration decisions
    - Update progress.md
    - /checkpoint:create 'phaseY-client-complete'

Output <promise>CLIENT_COMPLETE</promise> when the client connects to real backend with all features working and zero mock data remaining." --max-iterations {{CLIENT_MAX_ITER}} --completion-promise "CLIENT_COMPLETE"
```

---

## Phase D: Deployment & Hardening

> If `deployment` was a research question, this phase reads the resolved decision from Memory.

**Purpose**: Production-grade deployment with SSL, secrets, monitoring, backup, and streamlined startup.
**Depends on**: All feature + client phases
**Produces**: {{DEPLOYMENT_PRODUCES}}

```
/ralph-loop:ralph-loop "Execute {{PROMPT_FILE}} Phase D: Deployment & Hardening.

Read task_plan.md, findings.md, progress.md for context.
Read Memory: mcp__memory__search_nodes('Decision_') for resolved decisions (especially Decision_deployment).
Read Memory: mcp__memory__search_nodes('deployment') for prior context.

IMPLEMENTATION:

1. RESEARCH:
   {{BLOCK:DEPLOYMENT_RESEARCH}}
   - {{RESEARCH_TOOL}}: '{{RESEARCH_QUERY}}'
   {{/BLOCK:DEPLOYMENT_RESEARCH}}

2. PRODUCTION CONFIG:
   - Review current deployment configuration
   - Task(subagent_type='deployment-strategies:deployment-engineer') for hardening review
   {{BLOCK:DEPLOYMENT_CHECKLIST}}
   - {{DEPLOYMENT_ITEM}}
   {{/BLOCK:DEPLOYMENT_CHECKLIST}}

3. SSL/TLS:
   {{BLOCK:SSL_STEP}}
   - {{SSL_ITEM}}
   {{/BLOCK:SSL_STEP}}

4. SECRETS MANAGEMENT:
   - Generate script for all required secrets
   - Ensure no hardcoded secrets: mcp__serena__search_for_pattern('password|secret|api_key|token')
   - Task(subagent_type='comprehensive-review:security-auditor') for secrets review

5. MONITORING STACK:
   - Task(subagent_type='observability-monitoring:observability-engineer') for dashboards
   {{BLOCK:MONITORING_ITEM}}
   - {{MONITORING_STEP}}
   {{/BLOCK:MONITORING_ITEM}}

6. BACKUP & RESTORE:
   {{BLOCK:BACKUP_ITEM}}
   - {{BACKUP_STEP}}
   {{/BLOCK:BACKUP_ITEM}}

7. SECURITY HARDENING:
   - Task(subagent_type='comprehensive-review:security-auditor') final infrastructure audit
   - mcp__pal__codereview on deployment configs
   {{BLOCK:SECURITY_ITEM}}
   - {{SECURITY_STEP}}
   {{/BLOCK:SECURITY_ITEM}}

CHECKPOINT:
    - mcp__memory__create_entities for deployment architecture decisions
    - Update progress.md
    - /checkpoint:create 'phaseD-deployment-complete'

Output <promise>DEPLOYMENT_COMPLETE</promise> when {{DEPLOYMENT_DONE_CRITERIA}}." --max-iterations {{DEPLOYMENT_MAX_ITER}} --completion-promise "DEPLOYMENT_COMPLETE"
```

---

## Phase F: Testing & Validation (FINAL)

**Purpose**: Comprehensive testing to verify ALL success criteria. Final sign-off.
**Depends on**: All prior phases
**Produces**: All tests passing, load test results, security audit clean, success criteria checklist complete

```
/ralph-loop:ralph-loop "Execute {{PROMPT_FILE}} Phase F: Testing & Validation. FINAL PHASE.

Read task_plan.md, findings.md, progress.md for full context.
Read Memory: mcp__memory__read_graph() for all prior decisions and state.

COMPREHENSIVE TESTING:

1. UNIT TESTS:
   - Task(subagent_type='unit-testing:test-automator') for all services
   - Target: >80% code coverage per service

2. INTEGRATION TESTS:
   - Task(subagent_type='full-stack-orchestration:test-automator') for cross-service tests
   {{BLOCK:INTEGRATION_FLOW}}
   - {{FLOW_DESCRIPTION}}
   {{/BLOCK:INTEGRATION_FLOW}}

3. END-TO-END TESTS:
   - Task(subagent_type='full-stack-orchestration:test-automator') for E2E:
   {{BLOCK:E2E_SCENARIO}}
     - {{E2E_STEP}}
   {{/BLOCK:E2E_SCENARIO}}

4. LOAD TESTING:
   - Task(subagent_type='full-stack-orchestration:performance-engineer') for:
   {{BLOCK:LOAD_SCENARIO}}
     - {{LOAD_DESCRIPTION}}
   {{/BLOCK:LOAD_SCENARIO}}
   - Measure: latency p50/p95/p99, throughput, error rate

5. SECURITY TESTING:
   - Task(subagent_type='comprehensive-review:security-auditor') final audit
   - OWASP Top 10 checklist verification
   {{BLOCK:SECURITY_TEST}}
   - {{SECURITY_TEST_ITEM}}
   {{/BLOCK:SECURITY_TEST}}

{{BLOCK:DOMAIN_SPECIFIC_TEST}}
6. {{DOMAIN_TEST_TITLE}}:
   {{BLOCK:DOMAIN_TEST_STEP}}
   - {{DOMAIN_TEST_ITEM}}
   {{/BLOCK:DOMAIN_TEST_STEP}}
{{/BLOCK:DOMAIN_SPECIFIC_TEST}}

7. MONITORING VALIDATION:
   - Verify monitoring stack showing real data
   - Trigger test alert and verify delivery

8. FINAL CODE REVIEW:
   - /code-review on all changed files
   - /validate-and-fix for auto-fixable issues
   - mcp__pal__codereview: comprehensive final review
   - mcp__pal__challenge: '{{PROJECT_NAME}} is production-ready for {{TARGET_USERS}} {{GEO_DISTRIBUTION}} users'
   - Task(subagent_type='code-review-ai:architect-review') final architecture review

9. SUCCESS CRITERIA CHECKLIST (verify EVERY item):
    Functional:
    {{BLOCK:FUNCTIONAL_CHECK}}
    - [ ] {{FUNCTIONAL_CRITERION}}
    {{/BLOCK:FUNCTIONAL_CHECK}}

    Infrastructure:
    {{BLOCK:INFRA_CHECK}}
    - [ ] {{INFRA_CRITERION}}
    {{/BLOCK:INFRA_CHECK}}

    Deployment:
    {{BLOCK:DEPLOY_CHECK}}
    - [ ] {{DEPLOY_CRITERION}}
    {{/BLOCK:DEPLOY_CHECK}}

    Performance:
    {{BLOCK:PERF_CHECK}}
    - [ ] {{PERF_CRITERION}}
    {{/BLOCK:PERF_CHECK}}

10. FINAL CHECKPOINT:
    - mcp__memory__create_entities for: test results, benchmarks, security results, criteria status
    - Update progress.md with final status
    - Update task_plan.md marking all phases complete
    - /checkpoint:create 'production-ready'

Output <promise>PRODUCTION_READY</promise> when ALL success criteria checkboxes are verified, all tests pass, load tests meet targets, security audit is clean, and monitoring is operational." --max-iterations {{FINAL_MAX_ITER}} --completion-promise "PRODUCTION_READY"
```

---

## Quick Reference: Execution Sequence

```
COMMAND                                              PROMISE                    MAX ITER
──────────────────────────────────────────────────────────────────────────────────────────
Step 0: Bootstrap                                    BOOTSTRAP_COMPLETE         {{BOOTSTRAP_MAX_ITER}}
Phase 1: Assessment + Technology Selection           ASSESSMENT_COMPLETE        {{ASSESSMENT_MAX_ITER}}
Phase 2: {{PHASE_2_TITLE}}                           {{PHASE_2_PROMISE}}        {{PHASE_2_MAX_ITER}}
{{BLOCK:PHASE_ROW}}
Phase {{N}}: {{PHASE_TITLE}}                         {{PHASE_PROMISE}}          {{PHASE_MAX_ITER}}
{{/BLOCK:PHASE_ROW}}
Phase D: Deployment & Hardening                      DEPLOYMENT_COMPLETE        {{DEPLOYMENT_MAX_ITER}}
Phase F: Testing & Validation (FINAL)                PRODUCTION_READY           {{FINAL_MAX_ITER}}
{{IF_TEAM_MODE}}
Team Lead Orchestration                              ORCHESTRATION_COMPLETE     {{ORCHESTRATION_MAX_ITER}}
{{/IF_TEAM_MODE}}
──────────────────────────────────────────────────────────────────────────────────────────
                                          TOTAL MAX ITERATIONS:                {{TOTAL}}
{{IF_TEAM_MODE}}
                                          Parallel waves:                      {{PARALLEL_WAVE_COUNT}}
                                          Max concurrent teammates:            {{MAX_CONCURRENT}}
                                          Estimated token multiplier:          ~{{TOKEN_MULTIPLIER}}x
{{/IF_TEAM_MODE}}
```

## Max Iteration Guidelines

| Phase Type                           | Suggested Range | Notes                                         |
|--------------------------------------|-----------------|-----------------------------------------------|
| Bootstrap                            | 5               | Fixed setup, rarely needs more                 |
| Assessment (few research questions)  | 12-15           | Mostly gap analysis                            |
| Assessment (many research questions) | 18-22           | +3 iterations per open research field          |
| Foundation (APIs/Auth/Data)          | 20-25           | Many services to iterate through               |
| Feature implementation               | 12-20           | Depends on complexity                          |
| Critical path feature                | 20-25           | Extra room for research + iteration            |
| Client integration                   | 15-20           | Wiring up many touchpoints                     |
| Deployment & hardening               | 15-20           | Config-heavy but well-defined                  |
| Final testing & validation           | 20-25           | Many test categories                           |
| **Team Mode**                        |                 |                                                |
| Team lead orchestration              | 30-50           | Scales with number of waves                    |
| Per-wave overhead (create/shutdown)  | +3-5            | Team creation, task setup, shutdown per wave   |
| Competing hypotheses (critical path) | 15-20 per investigator | 2-3 investigators + 1 challenger      |

## Between Phases

### Sequential Mode

After each ralph-loop completes:

1. **Review output** - Check progress.md for what was accomplished
2. **Check findings** - Review findings.md for new discoveries or blockers
3. **Verify decisions** - `mcp__memory__search_nodes('Decision_')` to verify all tech decisions are stored
4. **Verify checkpoint** - Use `/checkpoint:list` to confirm checkpoint was saved
5. **Decide next** - Proceed to next phase, or re-run current phase if incomplete

### Team Mode — Between Waves

After all teammates in a wave finish:

1. **Check task list** - `TaskList()` to verify all tasks in the wave are `completed`
2. **Collect teammate findings** - Read Memory entities written by each teammate
3. **Review progress.md** - Each teammate updates this; check for conflicts or gaps
4. **Verify no file conflicts** - If teammates touched overlapping files, review diffs
5. **Resolve cross-phase issues** - If Teammate A's output affects Teammate B's work, resolve before next wave
6. **Shut down wave teammates** - `SendMessage(type='shutdown_request', ...)` for each
7. **Clean up team** - `TeamDelete()` to remove team resources
8. **Checkpoint** - `/checkpoint:create 'wave-{{N}}-complete'`
9. **Decide next** - Start next wave, or re-run failed phases with new teammates

### Team Mode — Handling Teammate Failures

If a teammate stops without completing its phase:

| Symptom | Action |
|---------|--------|
| Teammate goes idle, task incomplete | `SendMessage` with guidance to continue |
| Teammate errors repeatedly | Shut down, spawn replacement with adjusted prompt |
| File conflict between teammates | Pause wave, resolve manually, resume remaining teammates |
| Teammate drifts off-task | `SendMessage` with explicit redirect instructions |
| All teammates stuck on shared blocker | Lead investigates blocker, broadcasts solution |

## Cancelling a Stalled Loop

```
/ralph-loop:cancel-ralph
```

Then diagnose with:
- `mcp__memory__read_graph()` to see last saved state
- Read `progress.md` for last completed step
- Restart the phase with adjusted prompt if needed

## Dependencies Between Phases

### Dependency Graph

```
Step 0 ─── mandatory ──→ Phase 1
Phase 1 ── mandatory ──→ Phase 2  (all tech decisions must be resolved)
{{BLOCK:DEPENDENCY}}
{{FROM}} ── {{TYPE}} ──→ {{TO}}
{{/BLOCK:DEPENDENCY}}

Critical path: {{CRITICAL_PATH}}
Parallelizable: {{PARALLEL_PHASES}}
```

### Wave Mapping (Team/Hybrid Mode)

> Derive waves directly from the dependency graph. Phases with no inter-dependencies
> go into the same wave. Phases that depend on earlier phases go into later waves.

```
┌──────────┬──────────────────────────────────────────┬─────────┐
│  Wave    │  Phases                                  │  Mode   │
├──────────┼──────────────────────────────────────────┼─────────┤
│  Wave 0  │  Step 0, Phase 1                         │  solo   │
│  Wave 1  │  Phase 2                                 │  solo   │
{{BLOCK:WAVE_ROW}}
│  Wave {{WAVE_NUM}}  │  {{WAVE_PHASES}}              │  {{WAVE_MODE}}  │
{{/BLOCK:WAVE_ROW}}
│  Wave F  │  Phase F (Final Validation)              │  solo   │
└──────────┴──────────────────────────────────────────┴─────────┘

Teammate count per parallel wave:
{{BLOCK:WAVE_COUNT}}
  Wave {{WAVE_NUM}}: {{TEAMMATE_COUNT}} teammates ({{TEAMMATE_NAMES}})
{{/BLOCK:WAVE_COUNT}}

Total teammates across all waves: {{TOTAL_TEAMMATES}}
Estimated token multiplier: ~{{TOKEN_MULTIPLIER}}x vs sequential
```

### File Ownership Matrix (Team Mode)

> CRITICAL: Two teammates must NEVER edit the same file simultaneously.
> Define file ownership before spawning parallel teammates.

```
{{BLOCK:FILE_OWNERSHIP}}
Teammate: {{TEAMMATE_NAME}} (Phase {{N}})
  Owns: {{FILE_PATHS}}
  Reads (shared): {{SHARED_READ_PATHS}}
{{/BLOCK:FILE_OWNERSHIP}}
```

---

## Phase Design Checklist

### Core (All Modes)
- [ ] **Context read** - `Read task_plan.md, findings.md, progress.md` + Memory search
- [ ] **Decision read** - `mcp__memory__search_nodes('Decision_')` for resolved tech choices
- [ ] **Research step** - At least one of: Octagon, Brave, PAL apilookup
- [ ] **Serena code exploration** - `get_symbols_overview`, `find_symbol` before modifying
- [ ] **Implementation** - Using `replace_symbol_body`, `insert_after_symbol`, or agent tasks
- [ ] **Review** - `mcp__pal__codereview` or Task(security-auditor)
- [ ] **Testing** - TDD orchestrator or test-automator agent
- [ ] **Checkpoint** - Memory entity + progress.md update + `/checkpoint:create`
- [ ] **Promise** - Clear `<promise>NAME</promise>` with unambiguous done criteria

### Team Mode Additions
- [ ] **File ownership** - Phase only edits files it owns (no cross-teammate conflicts)
- [ ] **Task update** - `TaskUpdate(status='completed')` when phase promise fulfilled
- [ ] **Lead notification** - `SendMessage(type='message', recipient='team-lead', ...)` on completion
- [ ] **Shutdown readiness** - Teammate handles `shutdown_request` gracefully after completion
- [ ] **Standalone context** - Phase instructions include enough context to work without conversation history
- [ ] **Plan approval** - If `require_plan_approval: true`, teammate plans before implementing

## Agent Selection Guide

| Need                          | Agent / Tool                                              |
|-------------------------------|-----------------------------------------------------------|
| TDD for backend               | `backend-development:tdd-orchestrator`                    |
| FastAPI implementation         | `python-development:fastapi-pro`                          |
| Django implementation          | `python-development:django-pro`                           |
| Express/Node.js backend        | `javascript-typescript:javascript-pro`                    |
| Go backend                     | `systems-programming:golang-pro`                          |
| React/Next.js frontend         | `frontend-mobile-development:frontend-developer`          |
| Mobile (RN/Flutter)            | `frontend-mobile-development:mobile-developer`            |
| Security audit                 | `comprehensive-review:security-auditor`                   |
| Backend API security           | `backend-api-security:backend-security-coder`             |
| Frontend security              | `frontend-mobile-security:frontend-security-coder`        |
| Database optimization          | `database-cloud-optimization:database-optimizer`          |
| PostgreSQL schema design       | `database-design:sql-pro`                                 |
| Performance/load testing       | `full-stack-orchestration:performance-engineer`           |
| Deployment hardening           | `deployment-strategies:deployment-engineer`                |
| Kubernetes architecture        | `kubernetes-operations:kubernetes-architect`               |
| Terraform/IaC                  | `cloud-infrastructure:terraform-specialist`                |
| Observability/monitoring       | `observability-monitoring:observability-engineer`          |
| Network config review          | `observability-monitoring:network-engineer`                |
| Architecture review            | `code-review-ai:architect-review`                         |
| Cross-service integration tests| `full-stack-orchestration:test-automator`                  |
| Unit tests                     | `unit-testing:test-automator`                              |
| UX/UI review                   | `multi-platform-apps:ui-ux-designer`                      |
| Debugging                      | `error-debugging:debugger`                                 |
| Deep analysis                  | `mcp__pal__thinkdeep`                                      |
| Multi-model validation         | `mcp__pal__consensus`                                      |
| Assumption challenging         | `mcp__pal__challenge`                                      |
| Deep research                  | `mcp__octagon-deep-research-mcp__octagon-deep-research-agent` |
| API/library lookup             | `mcp__pal__apilookup`                                      |
| Web search                     | `mcp__brave-search__brave_web_search`                      |
| **Team Orchestration**         |                                                            |
| Team lead (coordinates only)   | Set `delegate_lead: true` in team config or use `mode='delegate'` in Task calls |
| Feature teammate (writes code) | `general-purpose` with `team_name` parameter               |
| Research teammate (read-only)  | `Explore` with `team_name` parameter                       |
| Security review teammate       | `comprehensive-review:security-auditor` with `team_name`   |
| Devil's advocate / challenger  | `Explore` with adversarial prompt                          |
| Plan-first teammate            | Any type with `mode='plan'` (requires lead approval)       |

---

# Appendix: Worked Examples

These three examples show the same template filled at different knowledge levels.

---

## Example A: "I just have a goal" (Maximum research)

A user who wants to build a collaborative document editor but hasn't chosen any technology yet.

```yaml
project_name: "CollabDocs"
project_slug: "collab-docs"
prompt_file: "PROMPT_COLLAB_DOCS.md"
goal_summary: "Build a production-ready collaborative document editor like Google Docs Lite for a 20-person team"

tech_stack:
  backend_framework:
    research: "What backend framework is best for real-time collaborative editing with OT/CRDT support?"
    constraints: ["Must handle WebSocket connections efficiently", "Good ecosystem for CRDT libraries"]
  backend_language:
    research: "What language has the best CRDT/OT library ecosystem for collaborative editing in 2026?"
  frontend_framework:
    research: "What frontend framework works best for a rich text editor with real-time collaboration?"
    constraints: ["Must integrate with ProseMirror or TipTap or similar"]
  database:
    research: "What database handles document versioning and real-time sync for collaborative editing?"
    constraints: ["Must support efficient diff storage", "Self-hostable"]
  cache:
    research: "Do we need a separate cache/pub-sub layer for real-time document sync?"
  deployment:
    value: "Docker Compose"  # User knows this one
  client_type:
    research: "Web app vs Electron vs both for a small team collaborative editor?"

services:
  research: "What service boundaries make sense for a collaborative document editor?"
  constraints: ["Separate auth from document logic", "Real-time sync as its own service"]

hosting:
  value: "self-hosted"
target_users:
  value: "20 concurrent editors"
geo_distribution:
  value: "single office LAN"

features:
  - name: "Real-Time Collaborative Editing"
    category: "crdt"
    critical_path: true
    approach:
      research: "Compare CRDT vs OT for a small-team document editor. Consider Yjs, Automerge, ShareDB."
      constraints: ["Must handle offline edits", "Must merge without conflicts"]
  - name: "Document Management"
    category: "documents"
    critical_path: false
    approach:
      research: "How should documents be stored, versioned, and organized? Flat files vs DB blobs vs object storage?"
  - name: "User Permissions"
    category: "permissions"
    critical_path: false
    approach:
      value: "Role-based: owner, editor, viewer per document"

success_criteria:
  functional:
    - value: "20 users simultaneously edit the same document without conflicts"
    - value: "Offline edits merge correctly when reconnecting"
    - value: "Document version history with restore"
    - research: "What other functional criteria are standard for collaborative editors?"
  infrastructure:
    - value: "SSL/TLS on all endpoints"
    - value: "Document backups automated"
  deployment:
    - value: "docker compose up starts everything"
  performance:
    - value: "Keystroke-to-screen latency < 100ms for remote users"
    - value: "Document sync within 500ms"
```

**Execution mode:** `"team"` — With 10 research fields AND 3 features, this project benefits most from:
- Phase 1: Competing hypotheses pattern for CRDT research (3 research teammates)
- Wave 2: Parallel feature implementation (3 feature teammates)

```yaml
execution_mode: "team"
team_config:
  team_name: "collab-docs-build"
  delegate_lead: true
  require_plan_approval: true
  default_teammate_model: "sonnet"
  waves:
    - wave: 0
      mode: "solo"
      phases: ["step0", "phase1"]              # 10 research questions resolved here
    - wave: 1
      mode: "solo"
      phases: ["phase2"]                       # Foundation
    - wave: 2
      mode: "parallel"
      phases: ["phase3", "phase4", "phase5"]
      teammates:
        - name: "crdt-dev"
          phase: "phase3"                      # Real-Time Collaborative Editing (CRITICAL)
          subagent_type: "general-purpose"
        - name: "docs-dev"
          phase: "phase4"                      # Document Management
          subagent_type: "general-purpose"
        - name: "perms-dev"
          phase: "phase5"                      # User Permissions
          subagent_type: "general-purpose"
    - wave: 3
      mode: "parallel"
      phases: ["phaseD", "phaseF-prep"]
      teammates:
        - name: "deploy-engineer"
          phase: "phaseD"
          subagent_type: "general-purpose"
        - name: "test-engineer"
          phase: "phaseF-prep"
          subagent_type: "general-purpose"
    - wave: 4
      mode: "solo"
      phases: ["phaseF"]
```

**What happens:** Phase 1 resolves ~10 research questions before any implementation begins (assessment iterations 20-22). Wave 2 runs the 3 feature phases in parallel as separate teammates, each with their own context and file ownership. The critical-path CRDT phase could optionally use the competing hypotheses pattern (Yjs vs Automerge vs ShareDB) before the main implementation teammate starts. Every later phase reads `Decision_*` entities from Memory.

---

## Example B: "I know some things, need research on others" (Mixed mode)

A user building an ML model serving platform who knows their backend stack but needs research on the ML-specific parts.

```yaml
project_name: "ModelServe"
project_slug: "model-serve"
prompt_file: "PROMPT_ML_PLATFORM.md"
goal_summary: "Build a production ML model serving platform with A/B testing, canary deploys, and monitoring"

tech_stack:
  backend_framework:
    value: "FastAPI"                           # Known
  backend_language:
    value: "Python"                            # Known
  frontend_framework:
    value: "React"                             # Known - admin dashboard
  database:
    value: "PostgreSQL"                        # Known - metadata store
  cache:
    value: "Redis"                             # Known - model result caching
  deployment:
    research: "Compare Kubernetes vs Docker Compose vs managed services (SageMaker, Vertex) for ML model serving"
    constraints: ["Must support GPU instances", "Must handle model version rollback"]
  client_type:
    value: "Web SPA"                           # Admin dashboard, known

services:
  value:
    - "auth"
    - "model-registry"
    - "inference"
    - "monitoring"
    - "ab-testing"

hosting:
  value: "AWS"
target_users:
  value: "500 req/sec inference, 10 admin users"
geo_distribution:
  value: "single region us-east-1"

features:
  - name: "Model Registry"
    category: "registry"
    critical_path: false
    approach:
      research: "Compare MLflow vs custom registry vs DVC for model versioning and artifact storage"
      constraints: ["Must track model lineage", "Must store large model files efficiently"]
  - name: "Inference Engine"
    category: "inference"
    critical_path: true
    approach:
      research: "Compare TorchServe vs Triton vs Ray Serve vs custom FastAPI for model inference at 500 req/sec"
      constraints: ["Must support PyTorch and TensorFlow", "Must handle GPU scheduling", "Batched inference"]
  - name: "A/B Testing & Canary"
    category: "ab-testing"
    critical_path: false
    approach:
      research: "How to implement traffic splitting for ML model A/B testing with statistical significance tracking?"
  - name: "Model Monitoring"
    category: "monitoring"
    critical_path: false
    approach:
      value: "Prometheus metrics + custom data drift detection using Evidently AI"

success_criteria:
  functional:
    - value: "Deploy a new model version with zero downtime"
    - value: "A/B test between two model versions with configurable traffic split"
    - value: "Automatic rollback if error rate exceeds threshold"
  infrastructure:
    - value: "GPU utilization monitoring and alerting"
    - value: "Model artifact versioned backup"
  deployment:
    - research: "What deployment criteria matter for ML serving in production?"
  performance:
    - value: "p95 inference latency < 100ms"
    - value: "500 req/sec sustained throughput"
    - value: "Model cold-start < 30 seconds"
```

**Execution mode:** `"hybrid"` — 5 research fields warrant team research in Phase 1. 4 features with 2 parallelizable pairs (registry+monitoring, inference+A/B after).

```yaml
execution_mode: "hybrid"
team_config:
  team_name: "model-serve-build"
  delegate_lead: true
  require_plan_approval: true
  waves:
    - wave: 0
      mode: "solo"
      phases: ["step0", "phase1", "phase2"]
    - wave: 1
      mode: "parallel"
      phases: ["phase3-registry", "phase3-monitoring"]
      teammates:
        - name: "registry-dev"
          phase: "phase3-registry"
          subagent_type: "general-purpose"
        - name: "monitoring-dev"
          phase: "phase3-monitoring"
          subagent_type: "general-purpose"
    - wave: 2
      mode: "parallel"
      phases: ["phase4-inference", "phase4-abtesting"]
      teammates:
        - name: "inference-dev"
          phase: "phase4-inference"
          subagent_type: "general-purpose"
        - name: "abtesting-dev"
          phase: "phase4-abtesting"
          subagent_type: "general-purpose"
    - wave: 3
      mode: "solo"
      phases: ["phaseD", "phaseF"]
```

**What happens:** Phase 1 resolves ~5 research questions (deployment, model registry, inference engine, A/B approach, deployment criteria). The known values (FastAPI, PostgreSQL, Redis, services list) pass straight through. Assessment needs ~16-18 iterations. Waves 1-2 run feature pairs in parallel.

---

## Example C: "I know everything, just execute" (Minimal research)

A user with a fully specified existing codebase (like the original voice-chat project).

```yaml
project_name: "Voice Chat"
project_slug: "voice-chat"
prompt_file: "PROMPT_PRODUCTION_READINESS_ASSESSMENT.md"
goal_summary: "Take existing voice-chat codebase to production readiness for self-hosted deployment"

tech_stack:
  backend_framework:
    value: "FastAPI"
  backend_language:
    value: "Python"
  frontend_framework:
    value: "React"
  database:
    value: "PostgreSQL"
  cache:
    value: "Redis"
  deployment:
    value: "Docker Compose"
  client_type:
    value: "Electron desktop"

services:
  value:
    - "auth"
    - "user"
    - "server"
    - "channel"
    - "message"
    - "file"
    - "webrtc"
    - "websocket"

service_base_path: "backend/services"
service_entry_file: "main.py"

hosting:
  value: "self-hosted"
target_users:
  value: "50-100 concurrent"
geo_distribution:
  value: "US geo-distributed"

features:
  - name: "Real-Time Messaging"
    category: "websocket"
    critical_path: false
    approach:
      value: "WebSocket gateway with Redis pub/sub for horizontal scaling"
  - name: "Voice Communication"
    category: "webrtc"
    critical_path: true
    approach:
      value: "mediasoup SFU with Coturn TURN server"
  - name: "File Uploads"
    category: "storage"
    critical_path: false
    approach:
      value: "MinIO with presigned URLs"

# All criteria are concrete -- no research needed
success_criteria:
  functional:
    - value: "50+ concurrent users connect from different locations"
    - value: "Text messaging with real-time WebSocket updates"
    - value: "Voice channels with clear audio across NAT via SFU"
    - value: "User auth with JWT refresh tokens"
    - value: "File uploads with validation and thumbnails"
  infrastructure:
    - value: "SSL/TLS on all services"
    - value: "Database backups automated"
    - value: "Prometheus + Grafana monitoring active"
  deployment:
    - value: "docker compose up -d starts everything"
    - value: ".env.example documented"
  performance:
    - value: "Voice latency < 200ms coast-to-coast"
    - value: "Message delivery < 500ms"
    - value: "SFU O(N) scaling for voice rooms"
```

**Execution mode:** `"sequential"` — Zero research questions, 3 features with tight dependencies (WebSocket feeds into WebRTC). No team overhead needed.

```yaml
execution_mode: "sequential"
# No team_config needed — all phases run as individual ralph-loops
```

**What happens:** Phase 1 has zero research questions to resolve. It only does gap analysis and assessment. Assessment needs only 12-15 iterations. This is equivalent to the original RALPH_LOOP_EXECUTION_GUIDE.md — the template degrades gracefully to the fully-specified case. No team infrastructure is created.

---

## Research Complexity Quick Reference

| Open research fields | Assessment max iterations | Phase 1 character | Recommended mode |
|----------------------|--------------------------|-------------------|-----------------|
| 0                    | 12-15                    | Pure gap analysis  | `sequential`    |
| 1-3                  | 15-18                    | Mostly execution   | `sequential`    |
| 4-7                  | 18-22                    | Research-heavy     | `hybrid`        |
| 8+                   | 22-25                    | Discovery project  | `team`          |

| Feature count | Dependencies | Recommended mode | Why |
|---------------|-------------|-----------------|-----|
| 1-2           | Any         | `sequential`     | Team overhead exceeds benefit |
| 3-4           | Tight chain | `sequential`     | Can't parallelize dependent phases |
| 3-4           | Some parallel| `hybrid`        | Parallel where possible |
| 5+            | Mostly independent | `team`    | Maximum throughput |

For every research field, expect ~2-3 tool calls (Octagon + Brave + PAL) plus one AskUserQuestion interaction for confirmation. In team mode, multiply token cost by ~Nx where N is the number of parallel teammates in the largest wave.
