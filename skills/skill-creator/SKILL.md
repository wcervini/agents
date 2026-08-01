---
name: skill-creator
description: Create and improve OpenCode skills (SKILL.md) and project rules files (AGENTS.md). Use when users want to create a skill from scratch, edit an existing skill, run evals to test a skill, benchmark skill performance with variance analysis, optimize a skill's description for better triggering accuracy, or create an AGENTS.md file for project-specific agent instructions. Covers the full OpenCode skill format (frontmatter validation, naming rules, permissions) and the AGENTS.md cross-agent standard.
---

# Skill & Rules Creator

Create skills (SKILL.md) and project rules (AGENTS.md) — the two main ways to give agents reusable instructions.

---

## Part 1: Creating AGENTS.md (Project Rules)

### What is AGENTS.md?

AGENTS.md is an open standard (used by 60k+ projects) for providing context and instructions to AI coding agents. It complements README.md — README is for humans, AGENTS.md is for agents.

**Cross-agent compatibility**: AGENTS.md works across OpenCode, OpenAI Codex, Google Jules, Cursor, Aider, Goose, Zed, Warp, VS Code, Devin, JetBrains Junie, GitHub Copilot, Windsurf, and many more.

### When to Create AGENTS.md

- Setting up a new project for AI-assisted development
- Migrating from Cursor rules, Copilot instructions, or Claude.md
- Adding project-specific conventions that agents should follow
- Documenting build, test, and lint commands for agent execution

### What to Include

Focus on what agents need but READMEs typically omit:

- **Build & test commands**: `pnpm dev`, `pnpm build`, `pnpm test`, `pnpm lint`
- **Command order**: e.g., "Run `pnpm lint` before committing"
- **Architecture**: project structure that isn't obvious from filenames
- **Code style**: TypeScript strict, naming conventions, import patterns
- **Security gotchas**: "Never commit .env files", "API keys go in secrets manager"
- **Deployment steps**: build → rsync → PM2 restart
- **Operational quirks**: "Database needs WAL mode enabled", "Nginx strips X-Forwarded headers"

### How to Create

**Option A: Use `/init` (OpenCode)**

Run `/init` in OpenCode — it scans project files, asks targeted questions, and creates/updates AGENTS.md automatically.

**Option B: Manual creation**

Create `AGENTS.md` at the project root:

```markdown
# Project Name

## Setup
- Install: `pnpm install`
- Dev: `pnpm dev`
- Build: `pnpm build`
- Test: `pnpm test`

## Code Style
- TypeScript strict mode
- Single quotes, no semicolons
- Functional patterns preferred
```

Commit AGENTS.md to Git — it's shared across the team and agents.

### File Locations & Precedence

OpenCode looks for rules in this order:
1. **Project**: `./AGENTS.md` (or `./CLAUDE.md` as fallback)
2. **Global**: `~/.config/opencode/AGENTS.md` (or `~/.claude/CLAUDE.md` as fallback)
3. **Nested**: subdirectory `AGENTS.md` closest to the edited file wins

The first matching file wins in each category. `AGENTS.md` takes precedence over `CLAUDE.md`.

### Referencing External Files

**Via opencode.json (recommended)**:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": [
    "CONTRIBUTING.md",
    "docs/guidelines.md",
    "packages/*/AGENTS.md"
  ]
}
```

**Via manual references in AGENTS.md**:

```markdown
For TypeScript guidelines: @docs/typescript-guidelines.md
For React patterns: @docs/react-patterns.md
```

Agents will read these files on a need-to-know basis.

### Monorepo Tip

Place AGENTS.md inside each subpackage. Agents read the nearest file in the directory tree, so every subproject ships tailored instructions.

---

## Part 2: Creating Skills (SKILL.md)

### What is a Skill?

A skill is a reusable set of instructions stored in `SKILL.md` with YAML frontmatter. Skills are loaded on-demand via the `skill` tool — agents see available skills and load the full content when needed.

### Skill File Structure

```
<skill-name>/
├── SKILL.md (required)
│   ├── YAML frontmatter (name, description required)
│   └── Markdown instructions
├── scripts/     - Executable code (optional)
├── references/  - Docs loaded on demand (optional)
└── assets/      - Templates/icons/fonts (optional)
```

### Skill Placement

OpenCode searches these locations (project-local first, global as fallback):
- `.opencode/skills/<name>/SKILL.md`
- `~/.config/opencode/skills/<name>/SKILL.md`
- `.claude/skills/<name>/SKILL.md` (Claude Code compat)
- `~/.claude/skills/<name>/SKILL.md`
- `.agents/skills/<name>/SKILL.md`
- `~/.agents/skills/<name>/SKILL.md`

### Frontmatter Requirements

Only these fields are recognized:

| Field | Required | Rules |
|-------|----------|-------|
| `name` | Yes | 1-64 chars, lowercase alphanumeric with single hyphens: `^[a-z0-9]+(-[a-z0-9]+)*$`. Must match directory name. |
| `description` | Yes | 1-1024 chars. Be specific enough for correct triggering. Include both what it does AND when to use it. |
| `license` | No | SPDX identifier (MIT, Apache-2.0, etc.) |
| `compatibility` | No | opencode, claude-code, etc. |
| `metadata` | No | String-to-string map (audience, workflow, tags, etc.) |

Unknown frontmatter fields are ignored.

### Name Validation

- 1–64 characters
- Lowercase alphanumeric with single hyphen separators
- Cannot start or end with `-`
- No consecutive `--`
- Must match directory name exactly

Valid: `git-release`, `astro-framework`, `my-skill-v2`
Invalid: `Git Release`, `--skill`, `skill_creator`, `my--skill`

### Description Tips

- Include trigger keywords and contexts the agent should recognize
- Be slightly "pushy" — agents tend to undertrigger skills
- Mention what happens if the skill is NOT used (negative triggers)

Example: *"Make sure to use this skill whenever the user mentions dashboards, data visualization, internal metrics, or wants to display any kind of company data, even if they don't explicitly ask for a 'dashboard'."*

### Skill Body Guidelines

- **Keep under 500 lines**. If approaching this limit, add references/ subdirectory with focused docs.
- **Progressive disclosure**: name+description (~100 words) always in context; SKILL.md loaded on trigger; references loaded on demand.
- **Imperative form**: "Run this command", "Use this pattern", not "You should consider running".
- **Explain the why**: LLMs work better when they understand the reasoning, not just rote MUSTs.
- **Include examples**: Show input/output pairs for clarity.
- **Reference files clearly**: "Load `references/deploy.md` when the user asks about deployment."

### Permission Configuration

Control skill access via `opencode.json`:

```json
{
  "permission": {
    "skill": {
      "*": "allow",
      "internal-*": "deny",
      "experimental-*": "ask"
    }
  }
}
```

| Permission | Behavior |
|------------|----------|
| `allow` | Skill loads immediately |
| `deny` | Hidden from agent, access rejected |
| `ask` | User prompted before loading |

Override per-agent in agent frontmatter or `opencode.json`.

---

## Part 3: The Creation Workflow

At a high level, the process goes:

- Decide what the skill/rule should do and roughly how
- Write a draft
- Create test prompts and run the agent with access to it
- Evaluate results (both qualitative and quantitative)
- Iterate based on feedback

### Capture Intent

Start by understanding what the user wants. Extract answers from the conversation:

1. What should this skill/rule enable the agent to do?
2. When should it trigger? (what user phrases/contexts)
3. What's the expected output format?
4. Should we set up test cases? Skills with objectively verifiable outputs (file transforms, code generation, fixed workflows) benefit from tests. Subjective skills (writing style, design) often don't.

### Interview and Research

Ask about edge cases, input/output formats, success criteria, dependencies. Check existing skills for patterns. Research via MCP tools or subagents if useful.

### Write the Draft

For **skills**, follow the SKILL.md structure above. For **AGENTS.md**, follow the markdown format from Part 1.

### Test Cases

After drafting, come up with 2-3 realistic test prompts. Share them with the user for approval.

Save test cases to `evals/evals.json`:

```json
{
  "skill_name": "example-skill",
  "evals": [
    {
      "id": 1,
      "prompt": "User's task prompt",
      "expected_output": "Description of expected result",
      "files": []
    }
  ]
}
```

### Running Evaluations (Skills Only)

**With-skill run**: spawn a subagent with the skill loaded.
**Baseline run**: same prompt without the skill (or with the old version).

Launch both in the same turn, organized in `<workspace>/iteration-N/`.

Capture `timing.json` from task notifications (tokens + duration).

### Grading & Benchmarking

1. Grade each run using assertions from `evals/evals.json`
2. Aggregate results:
   ```bash
   python -m scripts.aggregate_benchmark <workspace>/iteration-N --skill-name <name>
   ```
3. Launch viewer:
   ```bash
   nohup python <skill-creator-path>/eval-viewer/generate_review.py \
     <workspace>/iteration-N --skill-name "my-skill" \
     > /dev/null 2>&1 &
   ```

### Iteration Loop

1. Apply improvements based on feedback
2. Rerun test cases in new `iteration-N+1/`
3. Launch viewer with `--previous-workspace`
4. Get user feedback, improve again, repeat
5. Stop when user is happy, all feedback is empty, or no meaningful progress

### Description Optimization

After the skill works well, optimize the description for better triggering:

1. Generate 16-20 eval queries (mix of should-trigger and should-not-trigger)
2. Review with user
3. Run optimization loop:
   ```bash
   python -m scripts.run_loop \
     --eval-set <path-to-trigger-eval.json> \
     --skill-path <path-to-skill> \
     --model <model-id> \
     --max-iterations 5
   ```
4. Apply `best_description` to frontmatter

---

## Part 4: Cross-Environment Notes

### Claude.ai
- No subagents → run test prompts one at a time yourself
- Skip baselines and quantitative benchmarking
- Present results inline in conversation
- Skip description optimization (requires `claude -p`)

### Cowork
- Has subagents, so full workflow works
- No browser → use `--static <output_path>` for the viewer
- Run viewer BEFORE evaluating outputs yourself
- Description optimization works (uses `claude -p` via subprocess)

### Updating existing skills
- Preserve the original `name` field and directory name
- Copy to a writeable location first if installed path is read-only
- Package from the copy

---

## Reference files

- `agents/grader.md` — How to evaluate assertions against outputs
- `agents/comparator.md` — Blind A/B comparison
- `agents/analyzer.md` — Analyze benchmark results
- `references/schemas.md` — JSON structures for evals, grading, etc.

---

## Resources

- OpenCode Skills docs: https://opencode.ai/docs/skills/
- OpenCode Rules docs: https://opencode.ai/docs/rules/
- AGENTS.md standard: https://agents.md
- Keep a Changelog: https://keepachangelog.com/
- Semantic Versioning: https://semver.org/
