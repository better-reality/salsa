# Decision Log

## 2026-06-29 — OpenCode Agent Provisioning with Redirect Files

### Context
Executing point 4 from `.backlog/001-fast-sdlc-onboarding.md` - provisioning agents in OpenCode runtime environment.

### Approach
Instead of copying full files, created redirect files that contain instructions to read source files from `/.agents/`.

### Changes Made

1. **Created `.clinerules/` directory** — container for agent rules
2. **Created `.clinerules/workflows/` directory** — container for agent workflows
3. **Deployed 6 role redirect files** in `.clinerules/`:
   - `meta-agent.md` → `/.agents/rules/meta-agent.md`
   - `analyst.md` → `/.agents/rules/analyst.md`
   - `architect.md` → `/.agents/rules/architect.md`
   - `programmer.md` → `/.agents/rules/programmer.md`
   - `qa.md` → `/.agents/rules/qa.md`
   - `fast-guardian.md` → `/.agents/rules/fast-guardian.md`

4. **Deployed 10 workflow redirect files** in `.clinerules/workflows/`:
   - `meta-agent.md` → `/.agents/workflows/meta-agent.md`
   - `meta-agent-bootstrap-workforce.md` → `/.agents/workflows/meta-agent-bootstrap-workforce.md`
   - `analyst.md` → `/.agents/workflows/analyst.md`
   - `architect.md` → `/.agents/workflows/architect.md`
   - `programmer.md` → `/.agents/workflows/programmer.md`
   - `qa.md` → `/.agents/workflows/qa.md`
   - `qa-generate-test-suite.md` → `/.agents/workflows/qa-generate-test-suite.md`
   - `qa-execute-dynamic-testing.md` → `/.agents/workflows/qa-execute-dynamic-testing.md`
   - `qa-execute-non-functional-testing.md` → `/.agents/workflows/qa-execute-non-functional-testing.md`
   - `fast-guardian-fine-tune-agent.md` → `/.agents/workflows/fast-guardian-fine-tune-agent.md`

5. **Skipped MCP configuration** — will be done later per user request

### Rationale
- **Single Source of Truth:** Changes to `/.agents/` automatically reflect in OpenCode
- **No duplication:** No need to maintain two copies of rules
- **Human-readable:** Clear instructions in each redirect file
- **Portability:** Works across machines (no absolute paths)

### Files Created
- `/.clinerules/meta-agent.md`
- `/.clinerules/analyst.md`
- `/.clinerules/architect.md`
- `/.clinerules/programmer.md`
- `/.clinerules/qa.md`
- `/.clinerules/fast-guardian.md`
- `/.clinerules/workflows/*.md` (10 files)

### Notes
- MCP provisioning skipped (user decision)
- This approach is for human readers; OpenCode will need to follow the redirects manually or we may add auto-resolution later

## 2026-06-28 — Relative Paths for Portability

### Context
Absolute paths `/Users/sincerus/Workspace/code/external/better-reality/salsa` in configuration files prevented the project from working on other machines.

### Changes Made
Replaced absolute paths with relative `./` in 3 files:
- `/opencode.json` — MCP filesystem command
- `/DEVELOPMENT.md` — documentation example
- `/.agents/mcp/mcp-settings.json` — MCP server args

### Rationale
- `./` resolves to current working directory (project root)
- Works on any machine regardless of username or path structure
- Essential for team collaboration

### Files Modified
- `/opencode.json`
- `/DEVELOPMENT.md`
- `/.agents/mcp/mcp-settings.json`
- `/.agents/memory-bank/decision-log.md` — this entry

---

## 2026-06-28 — OpenCode Configuration Fix

### Context
Build mode was showing Kimi K2.5 instead of Minimax M2.1 in sessions. Also had duplicate MCP config in `.opencode/mcp-settings.json`.

### Root Cause
- `.opencode/mcp-settings.json` was not used by OpenCode (documentation doesn't mention it)
- Global config `~/.config/opencode/opencode.jsonc` lacked `agent` section
- Missing `agent` section in global config caused model fallback issues

### Changes Made

1. **Deleted** `.opencode/mcp-settings.json` — duplicate/unused
2. **Updated** `~/.config/opencode/opencode.jsonc` — added `agent` section with defaults:
   - `plan` → kimi-k2.5
   - `build` → minimax-m2.1
3. **Strategy**: Global config provides defaults, workspace `./opencode.json` overrides them

### Files Modified
- `/.opencode/mcp-settings.json` — deleted
- `~/.config/opencode/opencode.jsonc` — added agent section

### Testing Required
- Restart VS Code completely
- Verify Build Mode shows Minimax M2.1 in session
- Check other projects still work with their own opencode.json

---

## 2026-06-28 — MCP Configuration Migration

### Context
MCP configuration was in a separate file `mcp-config.json`, but OpenCode does not support external MCP configs via `configPath`.

### Changes Made

1. **Removed section** `"mcp": { "configPath": "..." }` from `opencode.json`
2. **Added section** `"mcpServers": { "filesystem": {...} }` to `opencode.json`
3. **Deleted file** `mcp-config.json`
4. **Updated instructions** in `/.clinerules/build.md` — role-based restrictions via instructions
5. **Updated documentation** in `DEVELOPMENT.md`

### Rationale
- OpenCode uses native `mcpServers` section
- Role-based restrictions implemented through agent instructions (not technical restrictions)
- Single configuration file is easier to maintain

### Files Modified
- `/opencode.json` — added `mcpServers` section
- `/.clinerules/build.md` — added safety rules
- `/DEVELOPMENT.md` — updated documentation
- `/.agents/memory-bank/decision-log.md` — this entry
- `/mcp-config.json` — deleted

---

## 2026-06-28 — MCP Protection for Build Agent

### Context
Need to protect Build agent from dangerous operations (rm -rf, deleting critical configs).

### Solution
Implemented MCP-based filesystem access with role-based restrictions.

### Configuration

**Created `/mcp-config.json`** — visible project-level MCP config:
- Filesystem MCP server with path restrictions
- Role-based permissions for Build agent

**Updated `/.clinerules/build.md`** — MCP usage rules:
- All filesystem operations MUST go through MCP
- Allowed without confirmation: read any file, write to `/src/`, `/tests/`, `/infra/`, `/.backlog/`
- Delete without confirmation: `/src/`, `/tests/` only
- Delete with confirmation: `/.agents/`, `/.clinerules/`, `/docs/`, `/infra/`
- STRICTLY FORBIDDEN: `/.agents/memory-bank/`, `/.clinerules.backup`

**Updated `/opencode.json`** — references MCP config:
- Added `"mcp": { "configPath": "./mcp-config.json" }`

### Files
- `/mcp-config.json` — MCP configuration with agentRoles
- `/.clinerules/build.md` — updated with MCP usage rules
- `/opencode.json` — references MCP config
- `/.opencode/mcp-settings.json` — simplified

### Rationale
- Prevents accidental deletion of critical project files
- Build agent can work safely in `/src/`, `/tests/`, `/infra/`, `/.backlog/`
- Dangerous operations require explicit human confirmation
- Memory bank and backups are protected from any modification

---

## 2026-06-28 — Unified Structure: OpenCode reads Cline files

### Context
We wanted to minimize duplication between OpenCode and Cline configurations.

### Solution
OpenCode now reads files directly from `/.clinerules/` instead of maintaining separate copies in `/.agents/rules/opencode/`.

### Changes Made

1. **Updated opencode.json:**
   - Plan: `/.clinerules/plan.md`, `/.clinerules/workflows/plan.md`
   - Build: `/.clinerules/build.md`, `/.clinerules/workflows/build.md`

2. **Deleted redundant directories:**
   - `/.agents/rules/opencode/` — removed
   - `/.agents/workflows/opencode/` — removed

3. **Result:** 4 files instead of 8 (50% reduction)

### Rationale
- Both OpenCode and Cline now use the same files from `/.clinerules/`
- Less maintenance, no duplication
- Single source of truth for Plan/Build agents

---

## 2026-06-28 — Native OpenCode Agents Structure

### Context
Needed to create unified agent structure for OpenCode (Plan/Build modes) and Cline with canonical sources in `/.agents/`.

### Solution

#### Adapter Structure
Created subdirectory `/.agents/rules/opencode/` with adapters:
- `plan.md` — adapter for OpenCode Plan Mode (based on meta-agent)
- `build.md` — adapter for OpenCode Build Mode (based on programmer)

Similarly for workflows: `/.agents/workflows/opencode/`

#### OpenCode Configuration
Updated `opencode.json` to load adapters in corresponding modes.

#### Cline Mirrors
Created `/.clinerules/` directory with runtime mirrors:
- `/.clinerules/plan.md` → mirror of opencode/plan.md
- `/.clinerules/build.md` → mirror of opencode/build.md
- `/.clinerules/workflows/` → workflow mirrors

Old file `.clinerules` renamed to `.clinerules.backup`.

### Files Created/Modified

**Adapters:**
- `/.agents/rules/opencode/plan.md` — Plan Mode adapter
- `/.agents/rules/opencode/build.md` — Build Mode adapter
- `/.agents/workflows/opencode/plan.md` — Plan workflow
- `/.agents/workflows/opencode/build.md` — Build workflow

**Cline Mirrors:**
- `/.clinerules/plan.md` — Cline mirror
- `/.clinerules/build.md` — Cline mirror
- `/.clinerules/workflows/plan.md` — Cline workflow mirror
- `/.clinerules/workflows/build.md` — Cline workflow mirror

**Config:**
- `/opencode.json` — updated configuration
- `/DEVELOPMENT.md` — documentation

**Backup:**
- `.clinerules.backup` — old file preserved

### Rationale
- OpenCode uses native Plan/Build modes
- Cline uses same structure via mirrors
- Canonical sources in `/.agents/` — single source of truth
- Adapters contain reduced versions + links to full rules

---

## 2026-06-28 — AI Model Configuration for Agents

### Context
During fast-asdlc onboarding to SALSA, we needed to define models for native OpenCode agents (plan/build).

### Options Considered

1. **Single model for all modes** (Kimi k2.5)
   - Pros: Simplicity, single context
   - Cons: Suboptimal cost and speed for routine coding

2. **Different models, global config** (`~/.config/opencode/`)
   - Pros: Works for all projects
   - Cons: Not in git, not reproducible on other machines

3. **Different models, per-workspace config** (`opencode.json`)
   - Pros: Versioned, reproducible, per-project flexibility
   - Cons: Requires creating file in each project

### Decision
Use per-workspace configuration (`opencode.json`) with separation:
- **Plan:** Kimi k2.5 (reasoning)
- **Build:** Minimax M2.1 (code)

### Rationale
- Minimax M2.1 showed good results in code generation tasks
- Separation optimizes costs (code-gen models are typically cheaper than reasoning)
- Per-workspace config ensures reproducibility for the team

### Related Files
- `/opencode.json` — Configuration
- `/AGENTS.md` — Agent instructions
- `/DEVELOPMENT.md` — Setup guide for humans
- `/README.md` — Updated with link to DEVELOPMENT.md

---

## 2024-06-28 — Fast-ASDLC Onboarding Kick-off
- **Decision:** Keep project discovery Markdown-only; do not select tech stack now.
- **Rationale:** Implementation is not imminent; architecture and analysis can proceed without stack lock-in.
- **Decision-maker:** Human supervisor.

- **Decision:** All agents run inside OpenCode; no Cursor dotfiles generated for now.
- **Rationale:** Current tooling environment is VS Code + OpenCode; other participants may use own setups.
- **Decision-maker:** Human supervisor.

- **Decision:** English-only for system artifacts; Russian permitted in `.backlog/` and `docs/domain/*Interview*.md`
- **Rationale:** Aligns with team bilingual operational reality while keeping code and specs portable.
- **Decision-maker:** Human supervisor.
